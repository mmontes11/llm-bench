# Running the benchmarks

How to produce the numbers recorded in [README.md](README.md). The README holds
**results only** — every instruction for generating them lives here.

## Reproducing (llama.cpp)

`bench.sh` in the [k8s-ai llama.cpp ConfigMap](https://github.com/mmontes11/k8s-ai/blob/main/infrastructure/llamacpp/llamacpp-configmap.yaml),
run by swapping the StatefulSet from `--server` to the benchmark command.

## Reproducing (vLLM)

Run from **inside the cluster, in the `ai` namespace**, with the target model Ready.
Substitute the two model-dependent values from this table; everything else is fixed so
runs stay comparable.

| model | `--served-model-name` / Service prefix | `--model` (tokenizer repo) |
|-------|----------------------------------------|----------------------------|
| nvidia | `nvidia-qwen-27b-nvfp4` | `nvidia/Qwen3.6-27B-NVFP4` |
| unsloth | `unsloth-qwen36-27b-nvfp4` | `unsloth/Qwen3.6-27B-NVFP4` |

### Target endpoint

KServe exposes each `LLMInferenceService` as a ClusterIP Service named
`<served-model-name>-kserve-workload-svc`, listening on **port 8000**, **plain HTTP**
(no TLS on the Service itself — the certs mounted at `/var/run/kserve/tls` front the
gateway, not this port). For the models above:

| model | base URL |
|-------|----------|
| nvidia | `http://nvidia-qwen-27b-nvfp4-kserve-workload-svc:8000` |
| unsloth | `http://unsloth-qwen36-27b-nvfp4-kserve-workload-svc:8000` |

Because the harness runs in the `ai` namespace, the short Service name resolves
directly — no port-forward, no `kubectl exec`, no FQDN needed. The fully qualified
`<svc>.ai.svc.cluster.local` form works identically if running from another namespace.

> [!WARNING]
> Do **not** benchmark through the external HTTPRoute hostname
> (`<model>.mmontes-internal.duckdns.org`). Every prefill figure in the README is derived
> as `input_len / TTFT`, and `tg128` comes from TPOT — per-token streaming latency. Both
> absorb ingress, TLS and WAN round-trip end to end. Measured from the same host:
> **2.8 ms** to the ClusterIP Service vs **117 ms** through the ingress. Results taken
> over the ingress are not comparable to the recorded tables, nor to the `llama-bench` rows.

### 1. Install the client, pinned to the server's vLLM version

```bash
uv tool install --python 3.12 'vllm==0.25.1' --with 'openai==2.45.0'
```

### 2. Confirm the target and record its context ceiling

```bash
curl -s http://nvidia-qwen-27b-nvfp4-kserve-workload-svc:8000/v1/models | \
  python3 -c 'import sys,json;d=json.load(sys.stdin)["data"][0];print(d["id"],d["max_model_len"])'
```

### 3. Confirm the card is not shared

The node advertises `nvidia.com/gpu: 10` via time-slicing, so a second pod can be
resident on the same physical GPU. Query DCGM directly rather than inferring from pod
state (`llamacpp` sleeps and releases VRAM after `--sleep-idle-seconds`, so a Running
pod does not imply resident VRAM):

```bash
curl -s http://nvidia-dcgm-exporter.gpu.svc.cluster.local:9400/metrics | \
  grep -E "^DCGM_FI_DEV_(FB_USED|FB_FREE|GPU_UTIL)\{"
```

A clean single-tenant card shows `FB_USED` ≈ `gpu-memory-utilization` × 23.5 GiB with
`GPU_UTIL 0` at idle — e.g. `FB_USED 22376`, `FB_FREE 1609` at `--gpu-memory-utilization=0.98`.

### 4. Run the three shapes

```bash
for SHAPE in "1024 512 8 2" "2048 128 6 1" "8192 128 6 1"; do
  set -- $SHAPE
  vllm bench serve \
    --backend vllm \
    --base-url http://nvidia-qwen-27b-nvfp4-kserve-workload-svc:8000 \
    --endpoint /v1/completions \
    --model nvidia/Qwen3.6-27B-NVFP4 \
    --served-model-name nvidia-qwen-27b-nvfp4 \
    --dataset-name random \
    --random-input-len "$1" \
    --random-output-len "$2" \
    --num-prompts "$3" \
    --num-warmups "$4" \
    --max-concurrency 1 \
    --ignore-eos \
    --seed 42 \
    --percentile-metrics ttft,tpot,itl,e2el
done
```

Expect roughly 8 minutes end to end at ~20 t/s: the `1024/512` shape dominates at
~25 s per request × 10 requests.

### 5. Capture the server-side conditions

These define the run as much as the client flags do.

```bash
kubectl -n ai logs -l app.kubernetes.io/name=nvidia-qwen-27b-nvfp4 -c main | grep -E \
  "Model loading took|Available KV cache memory|GPU KV cache size|Maximum concurrency|NVFP4 GEMM|marlin|uncalibrated"
```

Needs a kubeconfig with read access to the `ai` namespace, which the benchmark host
does not necessarily have — the client steps above only need Service DNS. The
`NVFP4 GEMM` term surfaces the selected FP4 kernel, logged as
`Using <Kernel> for NVFP4 GEMM`; its absence means every FP4 linear took the
pinned-Marlin `W4A16_NVFP4` path instead.

> [!IMPORTANT]
> - `--model` is the **tokenizer** source and must be a resolvable Hugging Face repo;
>   `--served-model-name` is what goes in the request payload. They are different values
>   and swapping them fails.
> - The `openai==2.45.0` pin is required. `uv tool install vllm==0.25.1` alone resolves
>   `openai==2.24.0`, which fails at import with `ImportError: cannot import name
>   'NamespaceTool' from 'openai.types.responses'`.
> - `--ignore-eos` is required, or output length depends on when the model stops and
>   tok/s becomes noise.
> - Only one model fits the 24 GiB card. Verify no other GPU workload is resident first
>   (step 3).

## Recording results in README.md

| README column | Where it comes from |
|---------------|---------------------|
| `tg128` | `1000 / Mean TPOT (ms)` — **not** `Output token throughput` |
| `pp2048` / `pp8192` | `input_len / Mean TTFT (s)` from the 2048 and 8192 runs |

`Output token throughput` divides generated tokens by wall-clock **including prefill**,
so it falls as prompts grow and is not comparable to `llama-bench`'s `tg128`. TPOT
excludes the first token and is flat across prompt lengths, which is the number wanted.

A vLLM result updates three places in the README: the model's own section (metric table,
`t/s` table, raw `vllm bench serve` table), any comparison table referencing it, and the
vLLM row in **Summary**. Keep derived figures consistent across all three.

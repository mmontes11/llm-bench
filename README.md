# LLM Benchmarks

LLM benchmarks using llama.cpp on Kubernetes.

### Benchmarks

-   [Qwen3.5-35B-A3B](#qwen35-35b-a3b)
-   [GPT-OSS 20B](#gpt-oss-20b)
-   [Gemma4 MXFP4 MoE](#gemma4-mxfp4-moe)

### Kubernetes Cluster

Provisioned by [k8s-management](https://github.com/mmontes11/k8s-management) and [k8s-infrastructure](https://github.com/mmontes11/k8s-infrastructure)

AI workloads and manifests (including llama.cpp) are managed in [k8s-ai](https://github.com/mmontes11/k8s-ai).

### Environment

**Infrastructure**: Kubernetes homelab

**GPU**: NVIDIA RTX PRO 4000 Blackwell SFF Edition
-   VRAM: 23.5 GiB
-   Compute Capability: 12.0
-   VMM: Enabled

**Benchmark Parameters**:
-   **Inference Engine**: llama.cpp build d00685831 (8660)
-   **CUDA Backend**: Enabled
-   **Flash Attention**: On
-   **Batch Size**: 2048
-   **UBatch Size**: 2048
-   **Threads**: 1
-   **Prompt Lengths**: 2048, 8192 tokens
-   **Tokens Generated**: 128

---

## Qwen3.5-35B-A3B

**Model**: Qwen3.5-35B-A3B Q4_K_Medium (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 20.09 GiB |
| Parameters | 34.66 B |
| Quantization | Q4_K_Medium |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2995.23 ± 11.71 |
| pp8192 (prompt processing) | 2608.68 ± 7.36 |
| tg128 (tokens generated) | 84.87 ± 0.28 |

---

## GPT-OSS 20B

### MXFP4 Quantization

**Model**: gpt-oss-20b MXFP4 MoE (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 11.27 GiB |
| Parameters | 20.91 B |
| Quantization | MXFP4 |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2703.86 ± 4.76 |
| pp8192 (prompt processing) | 1800.39 ± 5.17 |
| tg128 (tokens generated) | 109.68 ± 1.58 |

---

### F16 Quantization

**Model**: gpt-oss-20b F16 (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 12.83 GiB |
| Parameters | 20.91 B |
| Quantization | F16 |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2667.83 ± 4.94 |
| pp8192 (prompt processing) | 1785.77 ± 4.88 |
| tg128 (tokens generated) | 84.96 ± 0.75 |

---

## Gemma4 MXFP4 MoE

**Model**: gemma4 MXFP4 MoE (GGUF)
**Parameters**: 26B

| Metric | Value |
|--------|-------|
| Model Size | 15.52 GiB |
| Parameters | 25.23 B |
| Quantization | MXFP4 |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2841.20 ± 286.16 |
| pp8192 (prompt processing) | 2563.79 ± 1.95 |
| tg128 (tokens generated) | 77.33 ± 0.33 |

---

## Summary

| Model | Size | Params | PP2048 t/s | PP8192 t/s | TG128 t/s |
|-------|------|--------|------------|------------|-----------|
| Qwen3.5-35B-A3B Q4_K_Medium | 20.09 GiB | 34.66 B | 2995.23 | 2608.68 | 84.87 |
| GPT-OSS 20B MXFP4 MoE | 11.27 GiB | 20.91 B | 2703.86 | 1800.39 | 109.68 |
| GPT-OSS 20B F16 | 12.83 GiB | 20.91 B | 2667.83 | 1785.77 | 84.96 |
| Gemma4 MXFP4 MoE | 15.52 GiB | 26 B | 2841.20 | 2563.79 | 77.33 |

---

### Topics

[kubernetes](/topics/kubernetes "Topic: kubernetes") [benchmark](/topics/benchmark "Topic: benchmark") [llm](/topics/llm "Topic: llm") [llamacpp](/topics/llamacpp "Topic: llamacpp") [gpu](/topics/gpu "Topic: gpu") [cuda](/topics/cuda "Topic: cuda") [qwen](/topics/qwen "Topic: qwen") [gpt-oss](/topics/gpt-oss "Topic: gpt-oss") [moe](/topics/moe "Topic: moe") [quantization](/topics/quantization "Topic: quantization")

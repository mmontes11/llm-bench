# LLM Benchmarks

LLM benchmarks using llama.cpp on Kubernetes.

### Benchmarks

-   [Qwen3.6-27B](#qwen36-27b)
-   [Qwen3.6-27B MTP](#qwen36-27b-mtp)
-   [Qwen3.6-35B-A3B](#qwen36-35b-a3b)
-   [Qwen3.6-35B-A3B MTP](#qwen36-35b-a3b-mtp)
-   [Qwen3.5-27B](#qwen35-27b)
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
-   **Inference Engine**: llama.cpp build b76429a69 (8895) / 1738129be (9426) for MTP models
-   **CUDA Backend**: Enabled
-   **Flash Attention**: On
-   **Batch Size**: 2048
-   **UBatch Size**: 2048
-   **Threads**: 1
-   **Prompt Lengths**: 2048, 8192 tokens
-   **Tokens Generated**: 128

---

## Qwen3.6-27B

**Model**: Qwen3.6-27B Q4_K_Medium (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 15.65 GiB |
| Parameters | 26.90 B |
| Quantization | Q4_K_Medium |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 760.56 ± 4.58 |
| pp8192 (prompt processing) | 679.26 ± 5.65 |
| tg128 (tokens generated) | 18.31 ± 0.08 |

---

## Qwen3.6-27B MTP

**Model**: Qwen3.6-27B-MTP Q4_K_M (GGUF)
**Build**: 1738129be (9426)

| Metric | Value |
|--------|-------|
| Model Size | 15.92 GiB |
| Parameters | 27.32 B |
| Quantization | Q4_K_M |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 768.37 ± 5.34 |
| pp8192 (prompt processing) | 689.17 ± 6.34 |
| tg128 (tokens generated) | 18.57 ± 0.12 |

### Comparison: Qwen3.6-27B vs MTP

| Test | Qwen3.6-27B | Qwen3.6-27B MTP | Δ |
|------|-------------|-----------------|---|
| pp2048 | 760.56 | 768.37 | +1.0% |
| pp8192 | 679.26 | 689.17 | +1.5% |
| tg128 | 18.31 | 18.57 | +1.4% |

---

## Qwen3.6-35B-A3B

### MXFP4 MoE Quantization

**Model**: Qwen3.6-35B-A3B MXFP4 MoE (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 20.21 GiB |
| Parameters | 34.66 B |
| Quantization | MXFP4 |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2382.11 ± 13.14 |
| pp8192 (prompt processing) | 2112.82 ± 14.88 |
| tg128 (tokens generated) | 85.29 ± 0.21 |

### IQ4_NL Quantization (4.5 bpw)

**Model**: Qwen3.6-35B-A3B IQ4_NL (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 16.79 GiB |
| Parameters | 34.66 B |
| Quantization | IQ4_NL (4.5 bpw) |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2094.23 ± 10.83 |
| pp8192 (prompt processing) | 1869.25 ± 15.36 |
| tg128 (tokens generated) | 86.72 ± 0.05 |

### MXFP4 MoE MTP Quantization

**Model**: Qwen3.6-35B-A3B-MTP MXFP4 MoE (GGUF)
**Build**: 1738129be (9426)

| Metric | Value |
|--------|-------|
| Model Size | 20.65 GiB |
| Parameters | 35.51 B |
| Quantization | MXFP4 |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2531.76 ± 10.33 |
| pp8192 (prompt processing) | 2233.87 ± 17.27 |
| tg128 (tokens generated) | 95.93 ± 0.30 |

### Comparison: Qwen3.6-35B-A3B vs MTP (MXFP4 MoE)

| Test | Qwen3.6-35B-A3B MXFP4 MoE | Qwen3.6-35B-A3B-MTP MXFP4 MoE | Δ |
|------|---------------------------|-------------------------------|---|
| pp2048 | 2382.11 | 2531.76 | +6.3% |
| pp8192 | 2112.82 | 2233.87 | +5.7% |
| tg128 | 85.29 | 95.93 | +12.5% |

### IQ4_NL MTP Quantization (4.5 bpw)

**Model**: Qwen3.6-35B-A3B-MTP IQ4_NL (GGUF)
**Build**: 1738129be (9426)

| Metric | Value |
|--------|-------|
| Model Size | 17.25 GiB |
| Parameters | 35.51 B |
| Quantization | IQ4_NL (4.5 bpw) |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 2198.98 ± 16.39 |
| pp8192 (prompt processing) | 1942.82 ± 21.63 |
| tg128 (tokens generated) | 95.86 ± 0.15 |

### Comparison: Qwen3.6-35B-A3B vs MTP (IQ4_NL)

| Test | Qwen3.6-35B-A3B IQ4_NL | Qwen3.6-35B-A3B-MTP IQ4_NL | Δ |
|------|------------------------|----------------------------|---|
| pp2048 | 2094.23 | 2198.98 | +5.0% |
| pp8192 | 1869.25 | 1942.82 | +3.9% |
| tg128 | 86.72 | 95.86 | +10.5% |

---

## Qwen3.5-27B

**Model**: Qwen3.5-27B Q4_K_Medium (GGUF)

| Metric | Value |
|--------|-------|
| Model Size | 15.58 GiB |
| Parameters | 26.90 B |
| Quantization | Q4_K_Medium |

| Test | t/s |
|------|-----|
| pp2048 (prompt processing) | 787.63 ± 2.14 |
| pp8192 (prompt processing) | 708.77 ± 1.88 |
| tg128 (tokens generated) | 19.70 ± 0.07 |

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
| Qwen3.6-27B Q4_K_Medium | 15.65 GiB | 26.90 B | 760.56 | 679.26 | 18.31 |
| Qwen3.6-27B-MTP Q4_K_M | 15.92 GiB | 27.32 B | 768.37 | 689.17 | 18.57 |
| Qwen3.6-35B-A3B MXFP4 MoE | 20.21 GiB | 34.66 B | 2382.11 | 2112.82 | 85.29 |
| Qwen3.6-35B-A3B-MTP MXFP4 MoE | 20.65 GiB | 35.51 B | 2531.76 | 2233.87 | 95.93 |
| Qwen3.6-35B-A3B IQ4_NL | 16.79 GiB | 34.66 B | 2094.23 | 1869.25 | 86.72 |
| Qwen3.6-35B-A3B-MTP IQ4_NL | 17.25 GiB | 35.51 B | 2198.98 | 1942.82 | 95.86 |
| Qwen3.5-27B Q4_K_Medium | 15.58 GiB | 26.90 B | 787.63 | 708.77 | 19.70 |
| Qwen3.5-35B-A3B Q4_K_Medium | 20.09 GiB | 34.66 B | 2995.23 | 2608.68 | 84.87 |
| GPT-OSS 20B MXFP4 MoE | 11.27 GiB | 20.91 B | 2703.86 | 1800.39 | 109.68 |
| GPT-OSS 20B F16 | 12.83 GiB | 20.91 B | 2667.83 | 1785.77 | 84.96 |
| Gemma4 MXFP4 MoE | 15.52 GiB | 26 B | 2841.20 | 2563.79 | 77.33 |

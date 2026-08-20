# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Linux-x86_64` · llama.cpp `b10488`
CPU: **8 physical · 16 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 6.3 | 34% |
| 4 | 13.9 | 74% |
| 8 | 18.6 | 100% |
| 16 | 4.8 | 26% |
| 32 | 1.8 | 9% |

**Best**: `-t 8` at 18.6 tok/s
**Slowest tested**: `-t 32` at 1.8 tok/s (10.53x spread)
**Against the physical-core default** (`-t 8`, 18.6 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=8 make bench
```

## Your explanation

Knee nằm ở `-t 8`, đúng bằng 8 physical cores, với 18.6 tok/s. So với `-t 1`,
đây là speedup 2.95x. Khi tăng lên 16 rồi 32 threads, throughput giảm còn 4.8 và
1.8 tok/s: các thread bổ sung không tạo thêm memory bandwidth mà cạnh tranh cache,
băng thông và thời gian CPU, nên oversubscription làm decode chậm hơn.

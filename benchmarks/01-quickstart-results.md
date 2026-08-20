# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Linux-x86_64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3068 | 223 / 326 | 50.9 / 55.2 | 3405 / 3671 / 3671 | 19.7 |
| UD-Q2_K_XL | 2.24 | 3023 | 414 / 487 | 46.7 / 47.9 | 3223 / 3456 / 3456 | 21.4 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.09x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

UD-Q2_K_XL tiết kiệm 0.73 GB và decode nhanh hơn 1.09x (21.4 so với 19.7 tok/s),
nhưng TTFT P50 lại chậm hơn 1.86x (414 so với 223 ms). Vì vậy Q2 đáng dùng khi ưu
tiên giảm dung lượng và tăng throughput decode; Q4 hợp lý hơn nếu TTFT và chất lượng
trả lời quan trọng. Tôi chưa chạy phép so sánh cùng prompt để kết luận về chất lượng.

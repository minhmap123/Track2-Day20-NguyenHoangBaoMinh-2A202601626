# Bonus - Context-length sweep (prefill cost)

Host `Linux-x86_64` · llama.cpp `b10488` ·
`threads=8` `ngl=0` · RAM 15.5 GB

| Prompt tokens | Prefill (tok/s) | TTFT contribution (ms) | vs linear scaling |
|:--|--:|--:|--:|
| 256 | 119.0 | 2152.0 | 1.00x |
| 1024 | 113.4 | 9030.0 | 1.05x |

At 1024 tokens, prefill costs **9030 ms**, which is
**1.05x** linear scaling -- so on this hardware, over this range, prefill is
still growing **roughly linearly**, not quadratically.

That is the correct finding, not a failed experiment. Attention is O(N^2), but it is only
one term: the per-layer linear projections and MLP are O(N), and on a 2B-class model at
short prompts they dominate. The quadratic term only overtakes them once N gets large
enough. Your prefill cost is currently bounded by throughput, not by sequence length.

To find where it *does* bend, extend the grid:

```bash
.venv/bin/python bonus/sweeps/ctx-len-sweep.py --grid 1024,4096,8192,16384,32768
```

Watch the "vs linear" column: the first row that climbs meaningfully above 1.0 is where
attention starts to matter on your machine. Report that crossover point.

Either way, this is the number to remember when someone proposes stuffing more retrieved
context into a RAG prompt "because the context window allows it". Prefill is paid in full,
on every request, before the first token appears.

## Your finding

Trong range 256-1024 token, chưa thấy quadratic bend rõ: chi phí prefill ở 1024 token
là 1.05x mức tuyến tính. Tuy vậy throughput giảm từ 119.0 xuống 113.4 tok/s, tương
đương còn 0.95x, nên mỗi chunk được thêm vào vẫn làm TTFT tăng. Với RAG trên máy này,
mình nên giữ context gọn và chỉ lấy các chunk liên quan nhất.

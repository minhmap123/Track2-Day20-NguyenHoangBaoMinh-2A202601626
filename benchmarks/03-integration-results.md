# 03 - Integrate: RAG pipeline run

Host `Linux-x86_64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 3000.8 | 3000.9 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 2414.7 | 2414.7 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 2412.7 | 2412.8 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **2609.4** · total **2609.5**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, which removes the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

N16 Cloud/IaC, N17 Data pipeline, N18 Lakehouse và N19 Vector/features đều là stub
local/toy data; N19 dùng keyword overlap, chưa dùng vector index. N20 `llama-server`
là phần real. Dominant stage là LLM như dự kiến; để giảm latency một nửa, tôi sẽ
giảm output token hoặc tối ưu model/runtime vì embed và retrieve đều 0.0 ms.

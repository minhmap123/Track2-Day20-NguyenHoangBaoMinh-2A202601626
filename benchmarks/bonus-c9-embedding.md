# Bonus C9 - Embedding serving regime

I ran the repository's offline embedding demo because the real embedding server
would reuse the chat GGUF as a weak pooling-based encoder. The offline demo uses a
deterministic bag-of-words vector, with 8 corpus documents and one query.

| Rank | Cosine similarity | Result |
|:--|--:|:--|
| 1 | 0.572 | Embedding serving is prefill-bound: one forward pass, no KV cache, no decode loop. |
| 2 | 0.211 | PagedAttention stores the KV cache in non-contiguous virtual-memory pages. |
| 3 | 0.000 | Continuous batching lets requests join and leave the running batch every step. |

The result confirms that embedding serving is a different regime from chat serving:
there is no autoregressive decode loop or per-request KV cache, so throughput should
come from static batches and token sorting. This demo is only a logic baseline; a
production RAG system should use a dedicated encoder such as BGE-M3 or Qwen3-Embedding
instead of a pooled decoder model.
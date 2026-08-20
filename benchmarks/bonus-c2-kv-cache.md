# Bonus C2 - KV-cache quantization

I compared the default KV-cache type with `q8_0` for both K and V on the same
Gemma Q4 model, context size 2048, 8 threads and 4 slots. The same prompt and
`max_tokens=32` were used at temperature 0.

| Configuration | Response time | RSS observed | Output |
|:--|--:|--:|:--|
| Default KV cache | 1.91 s | about 1.9 GB | coherent |
| K/V `q8_0` | 1.92 s | about 4.0 GB | coherent |

The q8 run did not improve latency and used more observed process RSS in this
small context configuration. Both outputs were coherent, so this test found no
quality regression, but it was not a formal 10-prompt accuracy evaluation. RSS
is process-level and includes model/runtime allocations, so it should not be
read as KV-cache-only memory. The result still argues against enabling q8 by
default here; I would test larger context sizes before making a production choice.
# 02 - Serve: load test + saturation reading

Host `Linux-x86_64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 32 | 0.49 | 16000 | 23000 | 25000 | 7.3 | 0.0% |
| 50 | 40 | 0.63 | 34000 | 55000 | 56000 | 19.8 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.28x** (26% of linear) |
| P95 latency | **2.39x** |
| Effective concurrency at 50 users | 19.8 vs `--parallel 4` slots (occupancy/slot ratio 4.94) |

**Saturated.** Throughput delivered only 1.28x for 5x the offered load, and effective concurrency (19.8) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.28x while P95 moved 2.39x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

Server bão hòa ở mức tải 50 users: RPS chỉ tăng 1.28x khi offered load tăng 5x,
trong khi P95 tăng 2.39x lên 55 giây. Effective concurrency đạt 19.8 so với 4
slots và `requests_deferred` lên 46, nên phần latency tăng chủ yếu là queue time.
Để cải thiện goodput@SLO, tôi sẽ giảm output token hoặc tăng số slot nếu RAM cho phép;
đó là cách giảm thời gian chiếm slot trước khi tăng offered load.

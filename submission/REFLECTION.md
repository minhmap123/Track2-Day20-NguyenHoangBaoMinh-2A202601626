# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyễn Hoàng Bảo Minh
**MSSV:** 2A202601626
**Cohort:** A20-K4
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Linux 5.15.167.4-microsoft-standard-WSL2 (WSL2)
- **CPU:** 11th Gen Intel Core i7-11800H @ 2.30GHz
- **Cores:** 8 physical / 16 logical
- **CPU extensions:** AVX-512, AVX2
- **RAM:** 15.5 GB
- **Accelerator:** NVIDIA GeForce RTX 3050 Laptop GPU, 4 GB
- **llama.cpp asset đã tải:** llama.cpp b10488, CUDA build
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL

**Chạy ở đâu:** laptop local trong WSL2
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Mình chạy lab local trong WSL2. Sau `make setup`, mọi thứ hoạt động bình thường. Port
8080 bị Apache chiếm nên mình dùng port 8090 cho server và các lệnh client. Mình giữ
Gemma mặc định vì máy có đủ RAM.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3068 | 223 / 326 | 50.9 / 55.2 | 3405 / 3671 / 3671 | 19.7 |
| UD-Q2_K_XL | 2.24 | 3023 | 414 / 487 | 46.7 / 47.9 | 3223 / 3456 / 3456 | 21.4 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

UD-Q2_K_XL nhỏ hơn 0.73 GB và decode nhanh hơn 1.09x (21.4 so với 19.7 tok/s),
nhưng TTFT P50 lại chậm hơn 1.86x. Theo mình, Q2 phù hợp khi cần tiết kiệm dung lượng
và ưu tiên throughput; Q4 hợp lý hơn nếu cần TTFT tốt hơn. Mình chưa so sánh cùng một
prompt để kết luận chắc chắn về chất lượng trả lời.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.49 | 16000 | 23000 | 25000 | 7.3 | 0.0% |
| 50 | 0.63 | 34000 | 55000 | 56000 | 19.8 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.28x
- **P95 tăng:** 2.39x
- **Effective concurrency ở 50 users:** 19.8 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.95 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Peak mình đo được là 3.95/4 slots. LLM chiếm gần như toàn bộ latency pipeline; keyword
retrieval và embedding fallback gần như không tốn thời gian. Nếu muốn giảm latency 2x,
mình sẽ giảm số output token hoặc tối ưu model/runtime vì decode là bottleneck chính.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost only | stub |
| N17 Data pipeline | in-memory toy data | stub |
| N18 Lakehouse | toy dictionaries, no lakehouse service | stub |
| N19 Vector + features | keyword-overlap retrieval, no vector index | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 2609.4 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck, đúng như mình dự đoán vì model chạy local và decode chiếm gần như
toàn bộ 2609.5 ms tổng. Nếu cần giảm latency 2x, mình sẽ giảm số token sinh hoặc tối ưu
model/runtime; tối ưu keyword retrieval không tạo khác biệt đáng kể với toy corpus.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** Mình tăng số thread từ `-t 1` lên `-t 8`.

```
before:  -t 1 -> 6.3 tok/s
after:   -t 8 -> 18.6 tok/s
speedup: 2.95x
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Mình chọn `-t 8`, đúng bằng số physical cores. `-t 8` đạt 18.6 tok/s, cao hơn 2.95x
so với 6.3 tok/s ở `-t 1`. Throughput tăng vì decode tận dụng được các physical cores
để xử lý công việc song song.

Khi thử 16 và 32 threads, throughput giảm còn 4.8 và 1.8 tok/s. Decode bị giới hạn
bởi memory bandwidth và cache, nên các thread vượt quá số physical cores chỉ tranh chấp
tài nguyên và gây oversubscription. Vì vậy `-t 8` là điểm cân bằng tốt nhất trên máy mình.



---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** B2 context-length sweep, B4/C2 KV-cache quantization và B5/C9 embedding
serving regime ở chế độ offline.

**Numbers:**

**B2/B3 numbers:** context 256 token đạt `119.0 tok/s`, còn 1024 token đạt `113.4 tok/s`;
throughput còn `0.95x` khi context tăng 4x.

**B4/C2 numbers:** default KV cache mất `1.91 s` với RSS khoảng `1.9 GB`; K/V `q8_0`
mất `1.92 s` với RSS khoảng `4.0 GB`. Cả hai output đều coherent.

Demo offline xếp đúng tài liệu embedding serving ở đầu với cosine similarity `0.572`.

CUDA source build cho B1/C6 bị chặn ở bước configure vì CUDA Toolkit 12.0 từ chối
GCC 13.3; mình cũng đã thử ép GCC 12 nhưng CMake vẫn không hoàn tất compiler check.

**Điều này nói lên gì mà deck chưa nói:**

Context sweep cho thấy prefill ở 1024 token tốn `9030 ms`, nhưng mới chỉ là `1.05x`
scaling tuyến tính, chưa đủ dài để thấy rõ phần quadratic của attention. Vì vậy mình
không nên nhồi quá nhiều chunk vào RAG chỉ vì context window còn trống.

Với C2, q8 KV cache không cải thiện latency trong context 2048 và còn có RSS process
cao hơn ở lần đo này. Mình chưa coi RSS là dung lượng KV thuần túy, nhưng kết quả đủ để
không bật q8 mặc định nếu chưa có workload context dài chứng minh lợi ích.

Embedding serving là prefill-bound: mỗi text cần một forward pass, không có decode loop
hay KV cache như chat serving. Vì vậy throughput phù hợp với static batching thay vì
continuous batching. Demo dùng bag-of-words để minh họa logic, nên production vẫn cần
embedding model chuyên dụng như BGE-M3 hoặc Qwen3-Embedding.

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

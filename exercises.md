# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Overlap thấp do paraphrase hoặc lời hướng dẫn an toàn; human review xác nhận mọi claim đều có evidence. | Bịa thời hạn đổi trả, mức phí hoặc cam kết hoàn tiền không có trong corpus. | Đối chiếu từng claim với evidence; kiểm tra retrieved chunks và prompt trước khi sửa generation. |
| Answer Relevance | Từ chối đúng một yêu cầu ngoài scope hoặc prompt injection nên ít trùng từ với câu hỏi. | Trả lời về bảo hành khi khách hỏi hủy đơn, bỏ qua intent chính. | Review intent và tính đúng đắn của refusal; kiểm tra query retrieval và cách tổ chức câu trả lời. |
| Context Recall | Expected answer có từ đồng nghĩa nên overlap thấp, nhưng retrieved chunks vẫn chứa đủ điều kiện cần thiết. | Thiếu ngày hiệu lực hoặc ngoại lệ khiến chọn sai phiên bản chính sách đổi trả. | So sánh gold evidence với trace; đo lại sau khi điều chỉnh query, chunking hoặc top-k. |
| Context Precision | Có vài chunks dư nhưng đủ evidence và câu trả lời vẫn đúng; cần theo dõi chi phí và noise. | Chunk không liên quan đứng đầu, lấn át evidence đúng và gây trả lời sai. | Inspect thứ tự chunks; thử reranking trên cùng tập chunks và so sánh precision trước/sau. |
| Completeness | Bỏ phần diễn đạt dư trong reference nhưng vẫn đủ thông tin khách cần hành động. | Bỏ điều kiện thành viên phải active lúc đặt hàng hoặc không nêu phí restocking. | Review các facts bắt buộc; phân biệt thiếu evidence ở retrieval với bỏ sót khi generation. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Giữ nguyên question, evidence, rubric và cặp responses A/B. Condition 1 đặt A trước B; condition 2 đặt B trước A, che tên model và dùng cùng cấu hình judge. Chạy nhiều cặp và lặp mỗi condition; ánh xạ điểm về đúng response trước khi so sánh. Đo tỷ lệ đổi lựa chọn và chênh lệch điểm của cùng response khi ở vị trí đầu/cuối. Nếu judge ưu tiên vị trí đầu dù nội dung không đổi, đó là dấu hiệu position bias; dùng thứ tự ngẫu nhiên và chấm cả hai thứ tự để giảm ảnh hưởng.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Chấm theo facts được evidence hỗ trợ, các điều kiện/ngoại lệ cần thiết và bước hành động phù hợp. Không cộng điểm cho độ dài, lặp ý hay văn phong trang trọng; phạt claim không được hỗ trợ. Một câu trả lời ngắn đủ ý phải được điểm tương đương bản dài có cùng thông tin. Kiểm tra rubric bằng các cặp ngắn/dài giữ nguyên facts để phát hiện phần thưởng vô lý cho verbosity.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Judge có thể chấm nhất quán nhưng sai với tiêu chuẩn domain. Nhờ hai người chấm độc lập các case đại diện theo cùng rubric, giải quyết bất đồng rồi so sánh điểm judge với nhãn đã thống nhất. Review các case lệch lớn, chỉnh rubric và kiểm tra lại trên tập giữ riêng. Ẩn tên model, dùng responses từ nhiều model để kiểm tra self-preference; không dùng chính điểm judge làm ground truth. Calibration giúp ngưỡng quality gate phản ánh lỗi thực tế thay vì chỉ phản ánh sở thích của model.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | Block nếu average < 0.85 hoặc giảm > 0.05 so với baseline | Đề xuất ưu tiên tính đúng chính sách; mọi lỗi nghiêm trọng đã xác nhận như bịa quyền hoàn tiền hoặc vi phạm privacy phải block dù average cao. |
| Answer Relevance | Block nếu average < 0.80 hoặc giảm > 0.05 so với baseline | Trợ lý cần trả lời đúng intent; review riêng adversarial cases để không phạt refusal đúng chỉ vì overlap thấp. |
| Completeness | Block nếu average < 0.80 hoặc giảm > 0.05 so với baseline | Cần giữ đủ ngày, phí, điều kiện và ngoại lệ; một case bỏ sót điều kiện quan trọng đã xác nhận có thể block dù average đạt. |

Các ngưỡng trên là đề xuất ban đầu cho quality gate, cần calibrate bằng human labels trước khi dùng production. Chạy cùng bộ câu hỏi và cấu hình để so với baseline; nếu điểm giảm do biến động output hoặc giới hạn overlap, review trace trước khi kết luận. Chúng không thay đổi công thức metrics, `overall_score()` hay pass rule của lab (ba answer scores đều >= 0.5).

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Offline evaluation chạy trên golden dataset trước khi merge/deploy khi code, prompt, model hoặc retrieval thay đổi; dùng unit tests và benchmark regression để chặn lỗi. Online evaluation theo dõi mẫu hội thoại thực tế, lỗi, feedback và thay đổi phân phối sau triển khai, với dữ liệu nhạy cảm được loại bỏ. Human review dùng để calibrate judge, kiểm tra case điểm thấp hoặc bất đồng, và xác nhận lỗi chính sách, safety/privacy hay refusal mà heuristic không đánh giá đủ. Bổ sung những failures đã xác minh vào tập regression rồi đo lại ở vòng tiếp theo.

---

## Part 2 — Core Coding (9:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| M03 | Medium | `06_warranty_policy.md`, `07_repair_and_technical_support.md` | Kết hợp loại lỗi được bảo hành, thời hạn PulsePhone X, quy trình sau return window và điều kiện loaner. Cần cả hai nguồn để nêu đủ availability, identity verification và deposit USD 200; đây là kết hợp thông tin, chưa cần xử lý phiên bản chính sách. |
| H01 | Hard | `09_escalation_and_policy_updates.md` | Phải chọn version 1.0 theo ngày đặt hàng August 31 dù giao hàng sau September 1, rồi tính cửa sổ từ ngày confirmed delivery. OrbitPlus active không vượt được ngoại lệ đơn cũ: vẫn 21 ngày, không phải 45 ngày. Độ khó đến từ hai mốc thời gian và ngoại lệ membership. |
| A03 | Adversarial — false premise | `00_system_scope.md`, `08_accounts_privacy_and_security.md` | Câu hỏi giả định biết order number đồng nghĩa có quyền truy cập. Expected answer phải bác bỏ tiền đề, yêu cầu verified authorization theo policy và giữ giới hạn không tiết lộ dữ liệu khách khác hoặc xem live order. Scope evidence và privacy evidence cùng hỗ trợ hành vi này. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> Khó nhất là giữ đúng điều kiện và ngoại lệ khi câu trả lời kết hợp nhiều quy tắc. H01/H05 phân biệt ngày đặt hàng quyết định version với ngày giao hàng bắt đầu đếm return days; H02 phân biệt opened-device window và miễn restocking cho verified defect; H04 giữ ngoại lệ diagnostic fee đã được miễn trước shipment. H03 có phép tính USD 320 × 90% = USD 288, được suy ra từ giá và mã giảm hợp lệ trong question rồi đối chiếu minimum USD 300 sau discounts trong nguồn. Mỗi claim đã được đối chiếu với các contexts của chính QA; không dùng chính sách thực tế ngoài corpus. Các đoạn evidence được trích trực tiếp từ file nguồn, rút đến câu hoặc đoạn cần thiết, không sửa wording. Validator PASS xác nhận cấu trúc/provenance; việc review ngữ nghĩa và difficulty được thực hiện riêng bằng đối chiếu question, expected answer và evidence.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.

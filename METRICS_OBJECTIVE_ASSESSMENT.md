# Đánh giá các chỉ số hiện tại so với mục tiêu đề tài

## 1. Kết luận nhanh

- Pipeline từ preprocessing, training đến evaluation đã chạy đầy đủ trên PRONOSTIA.
- Data split theo bearing và official test được tách khỏi quá trình model selection.
- Mô hình đã tạo predictions, checkpoints, reports và plots có thể truy vết.
- Tuy nhiên, mục tiêu hiệu năng chính của đề tài hiện chưa đạt.
- Mục tiêu RMSE dưới 10% lifetime chưa đạt.
- Mô hình multimodal CNN-BiLSTM-Attention chưa vượt vibration-only baseline.
- Ablation hiện chưa chứng minh Attention có lợi.
- Transformer comparison chưa được thực hiện.
- `OPT2_06` là candidate tốt nhất trên development folds, nhưng đây là VibrationOnlyBaseline và chưa được chạy official test.

Tình trạng Objectives hiện tại:

| Trạng thái | Số lượng |
|---|---:|
| Achieved | 2 |
| Partially Achieved | 2 |
| Not Achieved | 3 |
| Not Yet Evaluated | 1 |

## 2. Chỉ số nào quan trọng nhất

### 2.1 Official endpoint RMSE hoặc lifetime-normalized RMSE

Đây là chỉ số quan trọng nhất để kết luận đề tài có đạt mục tiêu chính hay không.

- Official test yêu cầu dự đoán một RUL cho mỗi test bearing.
- RMSE phạt mạnh các dự đoán sai lớn.
- Yêu cầu đề tài ghi rõ mục tiêu RMSE dưới 10% tổng lifetime.
- Vì lifetime giữa các bearing khác nhau, normalized RMSE hoặc tỷ lệ RMSE trên lifetime dễ so sánh hơn RMSE tính bằng giây.

Kết quả hiện tại của proposed ensemble:

- Official Endpoint RMSE: `4,140.0s`.
- Báo cáo hiện tại ước tính tương đương khoảng `20-23%` operating duration.
- Mục tiêu: dưới `10%` lifetime.
- Đánh giá: **Not Achieved**.

### 2.2 IEEE PHM 2012 score

Đây là chỉ số quan trọng của official benchmark vì nó phạt bất đối xứng giữa dự đoán sớm và dự đoán trễ.

Trong implementation hiện tại:

- Score nằm trong `[0, 1]`.
- Giá trị càng cao càng tốt.
- Giá trị tốt nhất là `1`.

Tài liệu Objectives ban đầu lại ghi `PHM Score dưới 100, càng thấp càng tốt`. Hai định nghĩa này không cùng thang đo và không thể so sánh trực tiếp.

Vì vậy:

- Không được kết luận đạt mục tiêu chỉ vì `0.08465 < 100`.
- Cần thống nhất lại công thức và thang đo với giảng viên trước final report.
- Với công thức đang dùng, model có score cao hơn mới tốt hơn.

### 2.3 Official endpoint MAE

Endpoint MAE cho biết trung bình mỗi test bearing bị dự đoán sai bao nhiêu giây tại endpoint.

- Dễ giải thích hơn RMSE.
- Ít bị một vài sai số rất lớn chi phối hơn RMSE.
- Phù hợp để hỗ trợ kết luận, nhưng không thay thế mục tiêu RMSE dưới 10%.

### 2.4 Mean trajectory NRMSE trên development folds

Đây là chỉ số chính để chọn candidate trong giai đoạn optimization.

- Đánh giá toàn bộ RUL trajectory thay vì chỉ điểm cuối.
- Chuẩn hóa theo lifetime của từng bearing.
- Cho mỗi bearing trọng số ngang nhau.
- Tránh việc candidate chỉ học cách dự đoán gần 0 tại endpoint.

Đây là chỉ số đúng để chọn `OPT2_06`, nhưng chưa phải official test result.

### 2.5 Các chỉ số hỗ trợ

- Window MAE và Window RMSE: đánh giá toàn bộ windows, dùng để xem khả năng tracking trajectory.
- Across-fold standard deviation: đánh giá độ ổn định giữa các bearing.
- Accuracy within 10% và 20%: cho biết số endpoint nằm trong sai số cho phép.
- Missing-temperature subgroup metrics: kiểm tra robustness khi temperature bị thiếu.
- Operating-condition metrics: xem model có ổn định giữa OC1, OC2 và OC3 hay không.

## 3. Development results trước optimization

Development evaluation gồm 468 windows và 6 validation bearing endpoints.

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) |
|---|---:|---:|---:|---:|
| Naive Median | 6,043.4 | 8,241.8 | 6,401.7 | 6,460.8 |
| Random Forest | 6,859.8 | 8,930.6 | 2,553.8 | 3,197.3 |
| CNNBiLSTMAttentionRUL | 5,396.8 | 7,929.6 | 2,979.8 | 3,190.2 |
| MainModel_NoAttention | 5,100.7 | 7,680.1 | 2,552.0 | 3,072.2 |
| MainModel_NoModalityDropout | 5,702.3 | 8,191.5 | 4,089.0 | 4,783.9 |
| LateConcatBaseline | 5,248.4 | 7,420.4 | **1,326.4** | 1,960.6 |
| TemperatureOnlyBaseline | 5,570.1 | 7,740.8 | 5,839.7 | 6,011.6 |
| VibrationOnlyBaseline | **4,964.8** | **6,987.3** | 1,738.0 | **1,947.8** |

Nhận xét:

- VibrationOnlyBaseline có Window MAE, Window RMSE và Endpoint RMSE tốt nhất trong nhóm baseline ban đầu.
- LateConcatBaseline có Endpoint MAE thấp nhất.
- Proposed CNNBiLSTMAttentionRUL kém hơn VibrationOnlyBaseline ở cả bốn chỉ số.
- MainModel_NoAttention tốt hơn proposed model có Attention.
- TemperatureOnlyBaseline có kết quả yếu nhất trong các Deep Learning baselines.

Kết luận từ bảng này:

- Chưa chứng minh multimodal model tốt hơn single-modality model.
- Chưa chứng minh Attention cải thiện performance.
- Vibration đang là nguồn tín hiệu có giá trị dự báo chính.

## 4. Kết quả optimization trên development folds

Optimization dùng 12 screened candidates và xác nhận 3 finalists trên đủ 6 folds.

| Candidate | Family | Mean trajectory NRMSE | Mean trajectory NMAE | NRMSE std | Global Window RMSE (s) | Endpoint MAE (s) |
|---|---|---:|---:|---:|---:|---:|
| **OPT2_06** | **VibrationOnlyBaseline** | **0.28789** | **0.22602** | **0.09767** | **6,486.8** | **359.9** |
| OPT2_05 | VibrationOnlyBaseline | 0.31834 | 0.27172 | 0.14005 | 6,855.1 | 1,459.2 |
| OPT2_09 | CNNBiLSTMAttentionRUL | 0.34792 | 0.29597 | 0.10429 | 6,714.7 | 3,159.3 |

Đánh giá:

- `OPT2_06` là development winner vì có mean trajectory NRMSE thấp nhất.
- `0.28789` tương đương khoảng `28.789%` lifetime-normalized trajectory error.
- Nếu so trực tiếp với mục tiêu dưới 10%, kết quả này vẫn chưa đạt.
- Candidate tốt nhất vẫn là vibration-only, không phải multimodal architecture được đề xuất.
- Official test của `OPT2_06` chưa được thực hiện, nên chưa thể dùng candidate này để kết luận official performance.

Lưu ý quan trọng về Endpoint MAE `359.9s`:

- Development bearings chạy đến failure nên actual RUL tại endpoint bằng `0`.
- Endpoint error thấp có thể xuất hiện khi model dự đoán gần `0` ở cuối trajectory.
- Vì vậy, Endpoint MAE `359.9s` chỉ là diagnostic metric.
- Không được dùng riêng con số này để tuyên bố model đạt mục tiêu hoặc tốt hơn trên official test.

## 5. Official test results hiện tại

Official evaluation gồm 866 windows và 11 test bearing endpoints có actual RUL lớn hơn 0.

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) | IEEE PHM score |
|---|---:|---:|---:|---:|---:|
| Random Forest | **6,558.1** | **7,645.7** | 3,502.9 | 4,378.0 | 0.11422 |
| CNNBiLSTMAttentionRUL, seed 42 | 8,450.9 | 10,251.3 | 3,751.8 | 5,093.6 | **0.13292** |
| CNNBiLSTMAttentionRUL, seed 123 | 8,090.6 | 9,772.5 | **2,972.5** | **3,529.0** | 0.09949 |
| CNNBiLSTMAttentionRUL, seed 999 | 8,447.1 | 10,227.8 | 3,349.9 | 4,018.0 | 0.08955 |
| CNNBiLSTMAttentionRUL ensemble | 8,191.5 | 10,000.6 | 3,264.0 | 4,140.0 | 0.08465 |

Đánh giá proposed ensemble so với Random Forest:

- Endpoint MAE tốt hơn `238.9s`, từ `3,502.9s` xuống `3,264.0s`.
- Endpoint RMSE tốt hơn `238.0s`, từ `4,378.0s` xuống `4,140.0s`.
- Window MAE kém hơn `1,633.4s`.
- Window RMSE kém hơn `2,354.9s`.
- IEEE PHM score thấp hơn, `0.08465` so với `0.11422`; với công thức hiện tại, đây là kết quả kém hơn.

Do đó, mục tiêu vượt traditional baseline chỉ mới **Partially Achieved**.

## 6. Seed stability và tolerance

Kết quả ba official seeds:

| Metric | Mean | Sample std | Best | Worst |
|---|---:|---:|---:|---:|
| Window MAE (s) | 8,329.5 | 206.9 | 8,090.6 | 8,450.9 |
| Window RMSE (s) | 10,083.9 | 269.9 | 9,772.5 | 10,251.3 |
| Endpoint MAE (s) | 3,358.1 | 389.7 | 2,972.5 | 3,751.8 |
| Endpoint RMSE (s) | 4,213.5 | 800.4 | 3,529.0 | 5,093.6 |
| IEEE PHM score | 0.10732 | 0.02272 | 0.13292 | 0.08955 |

Nhận xét:

- Endpoint RMSE có sample standard deviation `800.4s`, cho thấy model còn khá nhạy với random seed.
- Seed 123 tốt nhất về Endpoint MAE và Endpoint RMSE.
- Seed 42 có IEEE PHM score cao nhất.
- Ensemble không tự động tốt nhất ở mọi metric.

Official endpoint tolerance:

| Model | Trong 10% | Trong 20% |
|---|---:|---:|
| Random Forest | 1/11, 9.09% | 1/11, 9.09% |
| Proposed seed 42 | 1/11, 9.09% | 2/11, 18.18% |
| Proposed seed 123 | 0/11, 0.00% | 1/11, 9.09% |
| Proposed seed 999 | 0/11, 0.00% | 0/11, 0.00% |
| Proposed ensemble | 0/11, 0.00% | 1/11, 9.09% |

Các tỷ lệ này còn thấp, nên chưa thể tuyên bố model có độ chính xác ổn định trên official test bearings.

## 7. Missing-temperature results

| Model | Subgroup | Endpoint MAE (s) | Endpoint RMSE (s) | IEEE PHM score |
|---|---|---:|---:|---:|
| Random Forest | Usable temperature | 3,677.2 | 4,592.8 | 0.05001 |
| Random Forest | Fully missing temperature | 3,038.3 | 3,745.6 | 0.28544 |
| Proposed ensemble | Usable temperature | 3,364.7 | 4,315.6 | 0.08592 |
| Proposed ensemble | Fully missing temperature | 2,995.4 | 3,630.5 | 0.08126 |

Kết luận đúng từ subgroup analysis:

- Model vẫn chạy bình thường khi temperature bị thiếu hoàn toàn.
- Không xuất hiện NaN hoặc Inf do missing temperature.
- Availability gating và mask đạt mục tiêu robustness về mặt kỹ thuật.
- Tuy nhiên, kết quả không chứng minh rằng có temperature sẽ làm model chính xác hơn.
- Vì vậy, mục tiêu này chỉ được đánh giá **Partially Achieved**.

## 8. Đánh giá từng Objective

| Objective | Trạng thái | Đánh giá |
|---|---|---|
| End-to-end multimodal RUL pipeline | **Achieved** | Pipeline preprocessing, training, prediction và evaluation đã hoàn chỉnh |
| Official test không leakage | **Achieved** | Model selection dùng development folds; official test được đánh giá sau khi freeze configuration |
| Missing-temperature robustness | **Partially Achieved** | Model xử lý được fully missing temperature, nhưng chưa chứng minh lợi ích accuracy của temperature |
| Vượt traditional baseline | **Partially Achieved** | Ensemble tốt hơn Random Forest ở endpoint errors nhưng kém hơn ở window errors và PHM score |
| Vượt single-modality baseline | **Not Achieved** | VibrationOnlyBaseline tốt hơn proposed multimodal model trên development validation |
| Chứng minh Attention có lợi | **Not Achieved** | No-Attention ablation tốt hơn proposed model có Attention |
| RMSE dưới 10% lifetime | **Not Achieved** | Official relative RMSE khoảng 20-23%; development winner trajectory NRMSE khoảng 28.8% |
| Transformer comparison | **Not Yet Evaluated** | Chưa có training và result cho Transformer |

## 9. Cách trình bày ngắn gọn với giảng viên

Có thể trình bày theo hướng sau:

- Phần pipeline và experimental protocol đã hoàn thành.
- Model đã được đánh giá bằng bearing-level cross-validation và independent official test.
- Mục tiêu robustness khi thiếu temperature đã được xử lý bằng mask và availability gating.
- Kết quả thực nghiệm hiện chưa chứng minh multimodal Attention tốt hơn vibration-only.
- Candidate tốt nhất trên development là `OPT2_06`, thuộc VibrationOnlyBaseline, với mean trajectory NRMSE `0.28789`.
- Official proposed ensemble đạt Endpoint MAE `3,264.0s` và Endpoint RMSE `4,140.0s`.
- Mục tiêu RMSE dưới 10% lifetime chưa đạt.
- Transformer comparison và official evaluation của `OPT2_06` là các phần còn thiếu.

## 10. Kết luận cuối

Nếu hỏi chỉ số nào quyết định đề tài đã đạt mục tiêu hay chưa, cần ưu tiên:

1. Official lifetime-normalized Endpoint RMSE.
2. Official IEEE PHM 2012 score sau khi thống nhất đúng công thức.
3. So sánh proposed multimodal model với vibration-only baseline.
4. Attention ablation.
5. Transformer comparison.

Theo các chỉ số hiện có:

- Phần triển khai hệ thống: đạt.
- Phần integrity và reproducibility: đạt.
- Phần robustness khi thiếu temperature: đạt về vận hành, chưa đạt về chứng minh improvement.
- Phần accuracy mục tiêu: chưa đạt.
- Phần chứng minh multimodal và Attention tốt hơn baseline: chưa đạt.
- Phần Transformer: chưa thực hiện.

Vì vậy, trạng thái đúng của đề tài hiện tại là: **đã hoàn thành pipeline và có preliminary results đáng tin cậy, nhưng chưa hoàn thành toàn bộ performance Objectives của đề tài**.

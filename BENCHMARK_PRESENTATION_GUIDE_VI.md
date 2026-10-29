# Mentor giải thích phần Benchmark cho sinh viên

## 1. Trước hết, em phải hiểu Benchmark dùng để làm gì

Benchmark không phải chỉ là đưa ra một bảng số rồi chọn dòng có số nhỏ nhất.

Mục đích của Benchmark là trả lời các câu hỏi:

- Main model có thực sự tốt hơn cách làm đơn giản không?
- Dùng cả vibration và temperature có tốt hơn chỉ dùng vibration không?
- Attention có thực sự giúp model tốt hơn không?
- Deep Learning có tốt hơn Random Forest dùng handcrafted features không?
- Model có hoạt động ổn định trên bearing chưa từng thấy không?
- Kết quả hiện tại đã đạt Objectives của đề tài chưa?

Vì vậy, dự án phải có nhiều models để đối chiếu:

- Naive Median là mốc thấp nhất.
- Random Forest đại diện cho traditional Machine Learning.
- VibrationOnlyBaseline kiểm tra hiệu quả khi chỉ dùng vibration.
- TemperatureOnlyBaseline kiểm tra hiệu quả khi chỉ dùng temperature.
- LateConcatBaseline là cách kết hợp hai modalities đơn giản.
- CNNBiLSTMAttentionRUL là main model của đề tài.
- MainModel_NoAttention kiểm tra Attention có đóng góp hay không.
- MainModel_NoModalityDropout kiểm tra tác dụng của modality dropout.

Nếu chỉ train main model rồi báo một con số, mình không biết con số đó tốt hay kém. Baselines và ablations chính là cơ sở để kết luận.

## 2. Phải phân biệt main model và model đang có kết quả tốt nhất

Main model của đề tài là:

`CNNBiLSTMAttentionRUL`

Kiến trúc chính gồm:

`Vibration CNN + Temperature Encoder + Missing-aware Fusion + BiLSTM + Attention + Regression Head`

Model có kết quả tốt nhất hiện tại trên development folds lại là:

`OPT2_06`, thuộc `VibrationOnlyBaseline`.

Hai khái niệm này không giống nhau:

- Main model là kiến trúc nghiên cứu mà đề tài muốn xây dựng và kiểm chứng.
- Development winner là model đang có metric tốt nhất trong thí nghiệm hiện tại.

Khi thầy hỏi “model chính là model nào?”, em trả lời `CNNBiLSTMAttentionRUL`.

Khi thầy hỏi “model nào đang cho kết quả tốt nhất?”, em phải nói `OPT2_06`, thuộc vibration-only, trên development validation.

Không được nói `OPT2_06` là main model của đề tài.

## 3. Cách hiểu bảng Development Benchmark

Development evaluation dùng 6 leave-one-bearing-out folds.

Em hiểu đơn giản là:

- Có 6 learning bearings.
- Mỗi lần giữ lại một bearing làm validation.
- Năm bearings còn lại dùng để train.
- Lặp lại cho đến khi cả 6 bearings đều từng được dùng làm validation.

Cách chia này quan trọng vì model phải được kiểm tra trên một bearing chưa xuất hiện trong training của fold đó. Nếu chia ngẫu nhiên theo windows, windows của cùng một bearing có thể nằm ở cả train và validation, gây data leakage.

Kết quả chính:

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) |
|---|---:|---:|---:|---:|
| CNNBiLSTMAttentionRUL | 5,396.8 | 7,929.6 | 2,979.8 | 3,190.2 |
| MainModel_NoAttention | 5,100.7 | 7,680.1 | 2,552.0 | 3,072.2 |
| LateConcatBaseline | 5,248.4 | 7,420.4 | **1,326.4** | 1,960.6 |
| TemperatureOnlyBaseline | 5,570.1 | 7,740.8 | 5,839.7 | 6,011.6 |
| VibrationOnlyBaseline | **4,964.8** | **6,987.3** | 1,738.0 | **1,947.8** |

Em đọc bảng như sau:

- MAE và RMSE càng thấp càng tốt.
- VibrationOnlyBaseline có Window MAE, Window RMSE và Endpoint RMSE tốt nhất trong nhóm models ban đầu.
- LateConcatBaseline có Endpoint MAE tốt nhất.
- Main model có kết quả kém hơn vibration-only.
- Model bỏ Attention lại tốt hơn model có Attention.
- Temperature-only có kết quả yếu, nghĩa là vibration đang mang phần lớn thông tin dự báo.

Kết luận đúng:

“Trong thí nghiệm hiện tại, nhóm chưa chứng minh được multimodal model tốt hơn vibration-only. Nhóm cũng chưa chứng minh được Attention mang lại improvement.”

Đây không phải là lỗi phải che giấu. Negative result vẫn là kết quả khoa học hợp lệ, miễn là thí nghiệm và cách đánh giá đúng.

## 4. Tại sao không được chọn model chỉ bằng Endpoint MAE

Development bearings là run-to-failure bearings. Ở acquisition cuối cùng, actual RUL bằng `0`.

Nếu chỉ dùng Endpoint MAE để chọn model, một model thường xuyên dự đoán gần `0` ở cuối trajectory có thể được đánh giá rất tốt, dù nó theo dõi toàn bộ quá trình suy giảm không chính xác.

Vì vậy, optimization hiện tại dùng `mean trajectory NRMSE` làm primary metric.

Chỉ số này:

- Đánh giá toàn bộ RUL trajectory.
- Chuẩn hóa error theo lifetime của từng bearing.
- Cho mỗi bearing trọng số ngang nhau.
- Không để bearing dài có quá nhiều windows lấn át bearing ngắn.

Đây là lý do `OPT2_06` được chọn bằng trajectory NRMSE, không phải chỉ bằng Endpoint MAE.

## 5. Cách hiểu bảng Optimization

Optimization đã screen 12 configurations và chọn 3 finalists để xác nhận trên đủ 6 folds.

| Candidate | Family | Mean trajectory NRMSE | Mean trajectory NMAE | NRMSE standard deviation |
|---|---|---:|---:|---:|
| `OPT2_06` | VibrationOnlyBaseline | **0.28789** | **0.22602** | **0.09767** |
| `OPT2_05` | VibrationOnlyBaseline | 0.31834 | 0.27172 | 0.14005 |
| `OPT2_09` | CNNBiLSTMAttentionRUL | 0.34792 | 0.29597 | 0.10429 |

`OPT2_06` thắng vì có mean trajectory NRMSE thấp nhất.

`0.28789` có thể hiểu là normalized trajectory error khoảng `28.8%`.

Mục tiêu của đề tài là RMSE dưới `10%` lifetime. Dù hai cách đánh giá không hoàn toàn giống nhau, kết quả `28.8%` cho thấy candidate hiện tại vẫn còn cách khá xa mục tiêu.

Endpoint MAE của `OPT2_06` là `359.9s`, nhưng em không được dùng riêng số này để tuyên bố model đã rất tốt. Actual RUL tại development endpoints bằng `0`, nên con số này chỉ là diagnostic metric.

Một điểm nữa phải nhớ: `OPT2_06` chưa được chạy official test. Vì vậy, mình chưa biết nó có generalize tốt trên 11 official test bearings hay không.

## 6. Cách hiểu bảng Official Test

Official test gồm 11 bearings độc lập. Khác với development endpoints, các test bearings bị dừng trước failure nên actual endpoint RUL lớn hơn `0`.

Vì vậy, Endpoint MAE, Endpoint RMSE và IEEE PHM score ở bảng này có ý nghĩa trực tiếp hơn đối với kết quả cuối của đề tài.

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) | IEEE PHM score |
|---|---:|---:|---:|---:|---:|
| Random Forest | **6,558.1** | **7,645.7** | 3,502.9 | 4,378.0 | 0.11422 |
| Main model, seed 42 | 8,450.9 | 10,251.3 | 3,751.8 | 5,093.6 | **0.13292** |
| Main model, seed 123 | 8,090.6 | 9,772.5 | **2,972.5** | **3,529.0** | 0.09949 |
| Main model, seed 999 | 8,447.1 | 10,227.8 | 3,349.9 | 4,018.0 | 0.08955 |
| Main model ensemble | 8,191.5 | 10,000.6 | 3,264.0 | 4,140.0 | 0.08465 |

Em cần nhìn bảng theo từng góc:

- Ensemble tốt hơn Random Forest ở Endpoint MAE và Endpoint RMSE.
- Random Forest tốt hơn ensemble ở Window MAE và Window RMSE.
- Random Forest cũng có IEEE PHM score cao hơn ensemble.
- Seed 123 tốt nhất về endpoint accuracy.
- Seed 42 tốt nhất về IEEE PHM score.
- Ensemble không phải lúc nào cũng tốt hơn từng single seed.

Vì vậy, không thể nói Deep Learning đã vượt Random Forest hoàn toàn.

Kết luận đúng là:

“Main model ensemble tốt hơn Random Forest ở endpoint errors nhưng kém hơn ở trajectory-level window errors và IEEE PHM score. Mục tiêu vượt traditional baseline mới chỉ đạt một phần.”

## 7. Phải hiểu đúng IEEE PHM Score

Trong implementation hiện tại:

- IEEE PHM score nằm trong `[0, 1]`.
- Càng cao càng tốt.
- Giá trị tốt nhất là `1`.

Tài liệu Objectives ban đầu lại ghi “PHM Score dưới 100, càng thấp càng tốt”. Đây là một thang đo khác.

Em không được nói:

“Model đạt yêu cầu vì `0.08465` nhỏ hơn `100`.”

Đây là cách so sánh sai.

Em nên nói:

“Công thức PHM trong implementation hiện tại dùng thang `[0, 1]`, càng cao càng tốt. Yêu cầu ban đầu ghi một định nghĩa khác, nên nhóm cần thống nhất lại công thức với giảng viên trước final report.”

## 8. Cách hiểu bảng Tolerance

Tolerance kiểm tra có bao nhiêu official endpoint predictions nằm trong sai số cho phép so với actual RUL.

| Model | Trong sai số 10% | Trong sai số 20% |
|---|---:|---:|
| Random Forest | 1/11, `9.09%` | 1/11, `9.09%` |
| Main model, seed 42 | 1/11, `9.09%` | 2/11, `18.18%` |
| Main model, seed 123 | 0/11, `0.00%` | 1/11, `9.09%` |
| Main model, seed 999 | 0/11, `0.00%` | 0/11, `0.00%` |
| Main model ensemble | 0/11, `0.00%` | 1/11, `9.09%` |

Proposed ensemble không có endpoint nào nằm trong sai số 10% và chỉ có 1 trong 11 endpoints nằm trong sai số 20%.

Điều này cho thấy model chưa dự đoán ổn định trên official test bearings.

## 9. Cách hiểu bảng Missing Temperature

Mục đích của bảng này không chỉ là xem metric nào thấp hơn. Trước hết, nó kiểm tra model có còn hoạt động khi temperature bị thiếu hoàn toàn hay không.

Kết quả proposed ensemble:

- Usable temperature:
  - Endpoint MAE: `3,364.7s`.
  - Endpoint RMSE: `4,315.6s`.
- Fully missing temperature:
  - Endpoint MAE: `2,995.4s`.
  - Endpoint RMSE: `3,630.5s`.

Kết luận mình được phép đưa ra:

- Model vẫn inference được khi temperature bị thiếu hoàn toàn.
- Không xuất hiện NaN hoặc Inf.
- Temperature mask và availability gating hoạt động đúng về mặt kỹ thuật.

Kết luận mình chưa được phép đưa ra:

- Không được nói temperature làm model kém đi.
- Không được nói bỏ temperature chắc chắn làm model tốt hơn.

Lý do là subgroup sizes nhỏ và đây chưa phải controlled experiment đủ để chứng minh quan hệ nhân quả.

Kết luận đúng là:

“Model đã đạt robustness về khả năng xử lý missing temperature, nhưng chưa chứng minh được temperature cải thiện accuracy.”

## 10. Đối chiếu với Objectives

| Objective | Trạng thái hiện tại | Vì sao |
|---|---|---|
| End-to-end multimodal pipeline | Achieved | Pipeline preprocessing, training, prediction và evaluation đã hoàn chỉnh |
| Independent official test | Achieved | Official test không được dùng để chọn model hoặc hyperparameters |
| Missing-temperature robustness | Partially Achieved | Model xử lý được missing temperature nhưng chưa chứng minh temperature tăng accuracy |
| Vượt traditional baseline | Partially Achieved | Tốt hơn Random Forest ở endpoint errors nhưng kém ở window errors và PHM score |
| Vượt single-modality baseline | Not Achieved | Vibration-only tốt hơn main multimodal model |
| Chứng minh Attention có lợi | Not Achieved | No-Attention ablation tốt hơn model có Attention |
| RMSE dưới 10% lifetime | Not Achieved | Official relative RMSE khoảng 20-23%; development winner trajectory NRMSE khoảng 28.8% |
| Transformer comparison | Not Yet Evaluated | Chưa có Transformer training và result |

## 11. Nếu thầy hỏi “vậy đề tài đã thành công chưa?”

Em không nên trả lời đơn giản là có hoặc không.

Em nên tách thành hai phần:

“Về implementation, pipeline đã hoàn thành. Nhóm đã có preprocessing, bearing-level validation, baselines, main model, ablation, checkpoints, predictions, official evaluation và reports.”

“Về performance Objectives, kết quả hiện chưa đạt đầy đủ. Main multimodal model chưa vượt vibration-only, Attention chưa cho thấy improvement, RMSE chưa dưới 10% lifetime và Transformer chưa được đánh giá.”

Đây là câu trả lời trung thực và đúng với evidence hiện có.

## 12. Phần em cần nhớ nhất

- Main model là `CNNBiLSTMAttentionRUL`.
- Development winner là `OPT2_06`, thuộc VibrationOnlyBaseline.
- Metric chính để chọn development candidate là mean trajectory NRMSE.
- `OPT2_06` đạt trajectory NRMSE `0.28789`, khoảng `28.8%`.
- `OPT2_06` chưa có official test result.
- Main model ensemble có Official Endpoint MAE `3,264.0s`.
- Main model ensemble có Official Endpoint RMSE `4,140.0s`.
- Mục tiêu RMSE dưới 10% lifetime chưa đạt.
- Main model chưa vượt vibration-only.
- Attention chưa được chứng minh có lợi.
- Missing-temperature handling hoạt động đúng, nhưng lợi ích accuracy của temperature chưa được chứng minh.
- Transformer comparison chưa thực hiện.

Nếu hiểu được các ý trên thì em có thể tự đọc các bảng trong `BENCHMARK.md` và `TRAINING_README.md`, đồng thời trả lời đúng phần lớn câu hỏi phản biện liên quan đến kết quả hiện tại.

# Mentor giải thích toàn bộ biểu đồ trong reports

Trong `reports/plots` hiện có 9 biểu đồ. Em cần phân biệt:

- Biểu đồ mô tả kết quả chính.
- Biểu đồ dùng để chẩn đoán model.
- Biểu đồ optimization hiện hành.
- Biểu đồ optimization cũ chỉ giữ làm historical evidence.

## 1. RUL trajectory của Bearing1_1

File: `reports/plots/rul_trajectory_bearing1_1.png`

### Biểu đồ thể hiện gì?

- Trục X: `Acquisition Index`, tức thứ tự các lần thu nhận tín hiệu.
- Trục Y: RUL tính bằng giây.
- Đường đen: Ground Truth RUL.
- Đường xanh: Random Forest.
- Đường đỏ: Main Model CNN-BiLSTM-Attention.

Ground Truth giảm gần tuyến tính từ khoảng 27.700 giây về 0 vì bearing đang tiến dần đến failure.

### Cách đọc

Model tốt phải tạo đường prediction:

- Gần đường Ground Truth.
- Giảm dần theo thời gian.
- Tiến gần 0 khi bearing tiến đến failure.

Kết quả hiện tại:

- Random Forest chủ yếu dự đoán quanh 5.000–7.000 giây trong phần lớn trajectory.
- Main model chủ yếu dự đoán quanh 4.000–5.000 giây.
- Cả hai đều dự đoán thấp hơn rất nhiều ở giai đoạn đầu.
- Main model không giảm về gần 0 ở cuối trajectory.
- Random Forest có giảm mạnh ở giai đoạn cuối nhưng vẫn không theo sát toàn bộ đường thật.

### Kết luận

Model đang có xu hướng dự đoán quanh một vùng trung bình thay vì theo dõi đầy đủ degradation trajectory.

Có thể nói:

> Trên Bearing1_1, cả Random Forest và main model đều chưa theo sát Ground Truth RUL. Main model thể hiện prediction compression, tức predictions tập trung trong một khoảng hẹp thay vì bao phủ toàn bộ lifetime.

Đây chỉ là một bearing minh họa. Không được dùng riêng hình này để kết luận model nào tốt nhất toàn bộ dataset.

## 2. Development endpoint scatter

File: `reports/plots/endpoint_scatter_comparison.png`

### Biểu đồ thể hiện gì?

- Trục X: Actual Endpoint RUL.
- Trục Y: Predicted Endpoint RUL.
- Điểm xanh: Random Forest.
- Điểm cam: Main model.
- Đường chéo `y = x`: prediction hoàn hảo.

Trong scatter plot bình thường:

- Điểm nằm trên đường chéo là dự đoán đúng.
- Điểm phía trên đường chéo là overprediction.
- Điểm phía dưới đường chéo là underprediction.

### Tại sao tất cả điểm nằm tại X = 0?

Sáu development bearings đều chạy đến failure. Vì thế:

- Actual RUL tại endpoint đều bằng `0`.
- Tất cả điểm có tọa độ X bằng `0`.
- Chỉ có Predicted RUL khác nhau trên trục Y.

### Ý nghĩa quan trọng

Biểu đồ này cho thấy vì sao Endpoint MAE không phù hợp để làm primary optimization metric trên development folds.

Nếu model luôn dự đoán gần 0 ở endpoint thì Endpoint MAE sẽ rất thấp, nhưng điều đó không chứng minh model theo dõi tốt toàn bộ RUL trajectory.

### Kết luận

> Development endpoint scatter bị suy biến vì mọi actual endpoint RUL đều bằng 0. Vì vậy, dự án phải dùng trajectory NRMSE thay vì chỉ dùng Endpoint MAE để chọn candidate.

Không được dùng hình này để đánh giá correlation giữa actual và predicted vì trục X không có variation.

## 3. Window-level absolute error boxplot

File: `reports/plots/error_distribution_boxplot.png`

### Boxplot thể hiện gì?

Mỗi box thể hiện phân phối absolute error của một model trên toàn bộ development windows.

- Đường màu cam trong box: median error.
- Mép dưới và trên của box: khoảng 25% đến 75% dữ liệu.
- Whiskers: phạm vi error còn lại theo quy tắc boxplot.
- Box càng thấp và càng ngắn thì model thường càng tốt và ổn định.

### Cách đọc

- `MainModel_NoAttention` có median error tương đối thấp.
- Main model có Attention có median error cao hơn No-Attention.
- Random Forest có box lớn và upper whisker rất cao, nghĩa là error phân tán mạnh và có nhiều trường hợp sai lớn.
- TemperatureOnlyBaseline có error tương đối cao.
- VibrationOnlyBaseline có phân phối error cạnh tranh hơn nhiều models còn lại.

### Lưu ý

Median trên boxplot không phải MAE.

- MAE là trung bình absolute error.
- Đường trong box là median absolute error.

Không được lấy chiều cao đường median để thay cho số MAE trong bảng.

### Kết luận

> Error distributions giữa các models còn rộng. No-Attention và vibration-only có phân phối error cạnh tranh, trong khi Random Forest có nhiều window errors rất lớn.

## 4. Modality ablation comparison

File: `reports/plots/modality_ablation_comparison.png`

### Biểu đồ thể hiện gì?

- Trục X: các model variants.
- Trục Y: endpoint error tính bằng giây.
- Cột xanh: Endpoint MAE.
- Cột đỏ: Endpoint RMSE.
- Cột càng thấp càng tốt.

### Kết quả quan trọng

- LateConcatBaseline có Endpoint MAE thấp nhất.
- VibrationOnlyBaseline có Endpoint RMSE thấp nhất.
- TemperatureOnlyBaseline có error cao nhất.
- No-Attention tốt hơn main model.
- No-ModalityDropout kém hơn main model.

### Cách hiểu

#### Vibration-only tốt

Vibration đang chứa phần lớn thông tin hữu ích cho RUL prediction.

#### Temperature-only kém

Temperature một mình chưa đủ để dự đoán RUL chính xác.

#### LateConcat tốt

Cách fusion đơn giản lại đạt kết quả tốt. Model phức tạp hơn không mặc nhiên tốt hơn.

#### No-Attention tốt hơn main model

Ablation hiện chưa chứng minh Attention mang lại improvement.

#### No-ModalityDropout kém hơn main model

Kết quả phù hợp với giả thuyết modality dropout có thể hỗ trợ robustness. Tuy nhiên, chưa nên khẳng định quan hệ nhân quả tuyệt đối nếu chưa có thêm controlled experiments.

### Kết luận

> Main multimodal architecture chưa vượt các baselines đơn giản. Kết quả hiện tại ủng hộ vibration-only và LateConcat hơn kiến trúc Attention đầy đủ.

## 5. Attention, fusion gate và temperature availability

File: `reports/plots/auxiliary_attention_and_gate_weights.png`

Hình này gồm ba biểu đồ con cho một sequence dài 32 time steps.

### Biểu đồ trên: Temporal Self-Attention

- Trục X: 32 acquisitions trong sequence.
- Trục Y: Attention weight.

Nếu Attention tập trung mạnh vào một vài time steps, weights sẽ có chênh lệch rõ.

Trong hình, weights chỉ nằm khoảng `0.0307` đến `0.0315`. Giá trị uniform của 32 time steps là:

`1 / 32 = 0.03125`

Điều đó nghĩa là Attention gần như phân bố đều trên toàn sequence. Model chỉ ưu tiên nhẹ phần giữa sequence.

Kết luận:

> Attention hiện chưa thể hiện khả năng tập trung mạnh vào một số degradation steps quan trọng.

Điều này phù hợp với kết quả No-Attention tốt hơn main model, nhưng chưa đủ để khẳng định Attention là nguyên nhân làm performance kém.

### Biểu đồ giữa: Cross-Modal Fusion Gate

Đây là effective gate kiểm soát mức đóng góp của temperature.

- Khi temperature chưa có, gate bằng 0.
- Khi temperature bắt đầu xuất hiện, gate tăng dần.
- Khi temperature có đầy đủ, gate ổn định quanh 0.51.

Điều này cho thấy model:

- Không sử dụng temperature khi dữ liệu chưa tồn tại.
- Tăng temperature contribution khi availability tăng.
- Không hiểu zero-filled missing values là temperature thật.

### Biểu đồ dưới: Temperature Availability

- Giá trị 0: temperature không có.
- Giá trị từ 0 đến 1: chỉ có một phần temperature history.
- Giá trị 1: temperature history đã đầy đủ.

Availability tăng từ 0 lên 1 ở khoảng time steps 7–14.

### Quan hệ giữa hai biểu đồ dưới

Fusion gate tăng gần đồng thời với availability. Đây là bằng chứng trực quan rằng missing-temperature policy hoạt động theo thiết kế.

### Điều không được kết luận

Không được nói:

- Temperature đóng góp chính xác 51% vào prediction.
- Temperature chắc chắn làm accuracy tăng.
- Attention đã tìm ra vùng degradation quan trọng.

Đây chỉ là auxiliary visualization của một sequence, không phải feature-importance analysis trên toàn dataset.

## 6. Official test endpoint scatter

File: `reports/plots/official_test_endpoint_comparison.png`

### Biểu đồ thể hiện gì?

- Trục X: Actual Ground Truth Endpoint RUL.
- Trục Y: Predicted Endpoint RUL.
- Điểm xanh: Random Forest.
- Điểm đỏ: CNN-BiLSTM-Attention ensemble.
- Đường chéo: perfect prediction.

Khác với development scatter, official actual RUL không bằng 0 nên biểu đồ này có ý nghĩa đầy đủ.

### Cách đọc

- Điểm càng gần đường chéo càng tốt.
- Điểm phía trên đường chéo: model dự đoán RUL quá cao.
- Điểm phía dưới đường chéo: model dự đoán RUL quá thấp.

### Quan sát chính

- Với một số bearings có actual RUL thấp, cả hai models dự đoán quá cao.
- Với một số bearings có actual RUL cao, models lại dự đoán quá thấp.
- `Bearing3_3` là trường hợp ensemble overpredict rất lớn.
- `Bearing1_7` và `Bearing2_3` bị underpredict lớn.
- Predictions bị tập trung trong phạm vi hẹp hơn actual RUL.

### Ý nghĩa

Đây tiếp tục là dấu hiệu prediction compression:

- Actual values trải rộng.
- Predictions không bao phủ tốt toàn bộ phạm vi.
- Model có xu hướng kéo predictions về vùng trung bình.

### Kết luận

> Official predictions còn cách xa perfect prediction line ở nhiều bearings. Model chưa generalize ổn định trên toàn bộ RUL range.

## 7. Official per-bearing signed error

File: `reports/plots/official_per_bearing_error.png`

### Signed error là gì?

`Signed Error = Predicted - Actual`

- Error dương: overprediction.
- Error âm: underprediction.
- Càng gần 0 càng tốt.
- Đường ngang nét đứt tại 0 là prediction hoàn hảo.

Mỗi bearing có:

- Cột xanh: Random Forest.
- Cột đỏ: main model ensemble.

### Quan sát chính

- `Bearing3_3`: ensemble overpredict khoảng 8.500 giây.
- `Bearing2_3`: cả hai models underpredict khoảng 5.500–5.800 giây.
- `Bearing1_7`: cả hai models underpredict mạnh.
- `Bearing2_4`: Random Forest overpredict hơn 8.000 giây, ensemble giảm error xuống khoảng 2.300 giây.
- `Bearing1_5` và `Bearing2_5`: ensemble cải thiện rõ so với Random Forest.
- Một số bearings ensemble tốt hơn, nhưng một số bearings lại kém hơn.

### Ý nghĩa

Một vài bearings có error cực lớn đang làm RMSE tăng mạnh.

Đây là lý do:

- RMSE lớn hơn MAE.
- Model có thể có MAE tương đối chấp nhận được nhưng RMSE vẫn rất cao.
- Cần xem per-bearing errors thay vì chỉ nhìn một số trung bình.

### Kết luận

> Ensemble không cải thiện đồng đều trên mọi bearing. Nó giảm error ở một số trường hợp nhưng tạo ra error rất lớn ở Bearing3_3.

## 8. Preliminary optimization comparison

File: `reports/plots/optimization_comparison.png`

### Đây là biểu đồ gì?

Đây là kết quả optimization cũ với namespace:

- `OPT_06`
- `OPT_02`
- `OPT_05`

Biểu đồ dùng:

- Endpoint MAE.
- Endpoint RMSE.
- Error bars theo fold.
- Đường ngang là LateConcat Endpoint MAE `1,326.4s`.

### Tại sao OPT_06 trông rất tốt?

`OPT_06` có endpoint error rất thấp vì development actual endpoint RUL đều bằng 0.

Model dự đoán gần 0 tại endpoint sẽ có score đẹp, dù trajectory prediction chưa chắc tốt.

### Quan trọng

Biểu đồ này là **superseded historical evidence**.

Không được dùng nó để nói:

- `OPT_06` là winner hiện tại.
- Optimization đã đạt kết quả rất tốt.
- Endpoint MAE thấp chứng minh model theo dõi RUL tốt.

### Cách trình bày

> Biểu đồ này thể hiện optimization protocol ban đầu. Protocol đó bị thay thế vì endpoint-at-zero tạo ra selection bias. Kết quả hiện hành phải dùng OPT2 và trajectory NRMSE.

Nếu báo cáo ngắn, không nên đưa hình này vào slide chính. Chỉ dùng khi cần giải thích tại sao nhóm sửa selection metric.

## 9. Corrected OPT2 optimization comparison

File: `reports/plots/optimization_v2_comparison.png`

Đây là biểu đồ optimization hiện hành và quan trọng hơn biểu đồ số 8.

### Biểu đồ bên trái: Stage 1 Screening

- Trục X: số trainable parameters, dùng log scale.
- Trục Y: mean trajectory NRMSE trên screening folds 0 và 2.
- Mỗi màu đại diện cho một model family.
- Điểm càng thấp càng tốt.

Các model families:

- Đỏ: CNNBiLSTMAttentionRUL.
- Xanh dương: LateConcatBaseline.
- Tím: MainModel_NoAttention.
- Xanh lá: VibrationOnlyBaseline.

### Cách đọc Stage 1

- `OPT2_06` có NRMSE thấp nhất.
- `OPT2_05` và `OPT2_09` là hai candidates tiếp theo.
- Model nhiều parameters hơn không tự động tốt hơn.
- Các vibration-only candidates nằm trong nhóm tốt nhất.

Ba candidates được chọn vào Stage 2:

- `OPT2_06`
- `OPT2_05`
- `OPT2_09`

### Biểu đồ bên phải: Stage 2 Confirmation

Ba finalists được chạy trên đủ 6 development folds.

Chiều cao cột:

- Mean trajectory NRMSE.
- Càng thấp càng tốt.

Error bar:

- Sample standard deviation giữa 6 folds.
- Error bar càng dài nghĩa là performance thay đổi nhiều giữa các bearings.

Kết quả:

- `OPT2_06`: `0.28789 ± 0.09767`.
- `OPT2_05`: `0.31834 ± 0.14005`.
- `OPT2_09`: `0.34792 ± 0.10429`.

### Kết luận

- `OPT2_06` là development winner.
- `OPT2_06` thuộc VibrationOnlyBaseline.
- Multimodal finalist `OPT2_09` kém hơn hai vibration-only finalists.
- Error bars khá lớn, cho thấy performance thay đổi đáng kể giữa các bearings.
- `OPT2_06` chưa có official test result.

Có thể nói:

> Sau khi sửa selection protocol, OPT2_06 đạt mean trajectory NRMSE thấp nhất trên 6 folds. Tuy nhiên, winner vẫn là vibration-only và normalized error khoảng 28.8%, nên mục tiêu dưới 10% chưa đạt.

## 10. Tóm tắt ý nghĩa của 9 biểu đồ

| Biểu đồ | Ý nghĩa chính |
|---|---|
| RUL trajectory | Model chưa theo sát toàn bộ degradation trajectory |
| Development endpoint scatter | Endpoint actual đều bằng 0 nên không phù hợp để chọn model |
| Error boxplot | Error distributions còn rộng và có nhiều sai số lớn |
| Modality ablation | Vibration-only và LateConcat tốt hơn main model |
| Attention and gate | Missing-temperature gate hoạt động; Attention gần uniform |
| Official endpoint scatter | Official predictions còn xa perfect line |
| Per-bearing signed error | Một vài bearings tạo ra error cực lớn |
| Preliminary optimization | Protocol cũ bị thay thế, không dùng làm kết quả hiện hành |
| OPT2 optimization | OPT2_06 là development winner theo trajectory NRMSE |

Thông điệp chung của toàn bộ plots:

> Pipeline và missing-temperature mechanism hoạt động đúng, nhưng main multimodal Attention model chưa chứng minh được lợi thế về accuracy. Vibration-only hiện vẫn là hướng tốt nhất trên development validation.

## 11. Nếu thầy hỏi nên đưa biểu đồ nào vào phần trình bày chính

Nên ưu tiên:

1. `rul_trajectory_bearing1_1.png` để minh họa trajectory prediction.
2. `modality_ablation_comparison.png` để trả lời đóng góp của từng architecture component.
3. `official_test_endpoint_comparison.png` để trình bày official test performance.
4. `official_per_bearing_error.png` để chỉ ra các bearings khó.
5. `optimization_v2_comparison.png` để giải thích candidate selection hiện hành.

Có thể để phần phụ:

- `error_distribution_boxplot.png`.
- `auxiliary_attention_and_gate_weights.png`.
- `endpoint_scatter_comparison.png`.

Không nên đưa vào phần kết quả chính:

- `optimization_comparison.png`, vì đây là optimization protocol cũ đã bị thay thế.

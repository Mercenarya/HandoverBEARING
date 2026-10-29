# BỐI CẢNH CHUNG CHO VÒNG TỐI ƯU TIẾP THEO

## 1. Mục tiêu của vòng này

Hai hướng sẽ được thử độc lập để trả lời hai câu hỏi:

1. Nếu tiếp tục phát triển hướng `Vibration-only` đang dẫn đầu thì chỉ số còn cải thiện được không?
2. Nếu sửa cách sử dụng `temperature`, `mask`, `fusion` và `attention` thì mô hình `Multimodal` có vượt được `Vibration-only` không?

Kết quả tốt hơn phải được xác định bằng development data. Không dùng official test để chọn mô hình.

## 2. Trạng thái hiện tại cần nhớ

### Hướng đang tốt nhất

`OPT2_06` là kết quả development tốt nhất hiện tại:

| Nội dung                                       |                 Giá trị |
| ----------------------------------------------- | ------------------------: |
| Model family                                    | `VibrationOnlyBaseline` |
| Mean trajectory NRMSE                           |               `0.28789` |
| Mean trajectory NMAE                            |               `0.22602` |
| Sample standard deviation của trajectory NRMSE |               `0.09767` |
| Global Window MAE                               |              `4279.7 s` |
| Global Window RMSE                              |              `6486.8 s` |

### Mốc Multimodal hiện tại

`OPT2_09` là mốc cần vượt của hướng Multimodal:

| Nội dung                                       |                 Giá trị |
| ----------------------------------------------- | ------------------------: |
| Model family                                    | `CNNBiLSTMAttentionRUL` |
| Mean trajectory NRMSE                           |               `0.34792` |
| Mean trajectory NMAE                            |               `0.29597` |
| Sample standard deviation của trajectory NRMSE |               `0.10429` |
| Global Window MAE                               |              `4936.6 s` |
| Global Window RMSE                              |              `6714.7 s` |

`OPT2_06` là mốc chung mà cả hai hướng phải cố gắng vượt.

## 3. Chỉ số dùng để chọn mô hình

Thứ tự ưu tiên cố định trước khi chạy:

1. Mean trajectory NRMSE thấp hơn.
2. Nếu chênh lệch NRMSE không quá `0.005`, chọn mean trajectory NMAE thấp hơn.
3. Nếu vẫn chưa phân biệt được, chọn global Window RMSE thấp hơn.
4. Sau đó mới xét sample standard deviation của trajectory NRMSE thấp hơn.
5. Cuối cùng mới xét số lượng parameters thấp hơn.

Endpoint MAE và Endpoint RMSE trên development chỉ dùng để quan sát. Không dùng chúng làm chỉ số chọn chính vì actual RUL tại các development endpoint đều bằng `0`.

Không thay đổi thứ tự chọn sau khi đã nhìn thấy kết quả.

## 4. Giao thức chung bắt buộc

- Giữ nguyên preprocessing, cache identity, target definition và bearing-level splits đã được data gate xác nhận.
- Screening dùng cùng development folds đã dùng ở vòng OPT2: folds `0` và `2`.
- Finalist phải chạy đủ cả 6 development folds: `0, 1, 2, 3, 4, 5`.
- Hai hướng phải dùng cùng epoch budget, early-stopping rule và seed policy.
- Normalization statistics chỉ được tính từ training bearings của fold tương ứng.
- Lưu prediction ở row level để notebook `04` có thể tính lại metrics độc lập.
- Mọi metric công bố phải được tạo ra từ notebook đã thực thi.
- Global RMSE phải tính từ toàn bộ row-level squared errors; không lấy trung bình RMSE của các folds.
- Official evaluation phải giữ `false` trong toàn bộ vòng tối ưu.
- Không load official prediction table, official label, official metric hoặc official plot trong đường chạy tối ưu.

## 5. Ngân sách thử nghiệm

Tổng ngân sách của vòng mới:

- Tối đa 6 screened configurations cho hướng Vibration-only.
- Tối đa 6 screened configurations cho hướng Multimodal.
- Tối đa 3 confirmed finalists cho cả hai hướng cộng lại.
- Mỗi finalist phải chạy đủ 6 development folds.

Hai bạn phải chốt bảng cấu hình trước khi xem kết quả của vòng mới. Không được bổ sung cấu hình theo kiểu chạy đến khi có chỉ số đẹp.

## 6. Namespace và bảo toàn bằng chứng

- Hướng Vibration-only dùng namespace `OPT3V_01` đến `OPT3V_06`.
- Hướng Multimodal dùng namespace `OPT3M_01` đến `OPT3M_06`.
- Không xóa, đổi tên hoặc ghi đè bất kỳ artifact `OPT_` hay `OPT2_` nào.
- Mọi checkpoint, history, manifest, prediction và report mới phải mang run ID mới.
- Phải ghi config hash, cache identity, split identity, fold, seed, source version và checkpoint identity.

## 7. Cách hai bạn làm song song

Mỗi bạn được giao một thư mục riêng và chỉ làm việc trong thư mục đó.

Cách làm an toàn:

1. Bạn chỉ sửa các file nằm trong thư mục được giao.
2. Bạn chỉ triển khai đúng hướng tối ưu trong file nhiệm vụ của mình.
3. Không sao chép kết quả hoặc artifact từ thư mục của bạn còn lại để làm kết quả của mình.
4. Giữ nguyên namespace được giao để hai bộ kết quả không bị nhầm lẫn.
5. Sau khi nhận đủ hai thư mục, người phụ trách sẽ đối chiếu và tích hợp kết quả vào bản chính.
6. Notebook `04_evaluation_and_ablation.ipynb` trong mỗi thư mục phải tính lại metrics từ prediction CSV của chính thư mục đó.

## 8. Quy tắc kiểm soát thay đổi khi dùng AI

Đây là bài toán tối ưu một pipeline đã chạy được, không phải xây lại một hệ thống mới. Vì vậy, mọi thay đổi phải nhỏ, có mục đích và truy vết được.

- Chỉ sửa phần code trực tiếp phục vụ giả thuyết đang thử.
- Không refactor diện rộng, đổi tên hàng loạt hoặc định dạng lại toàn bộ notebook.
- Không để AI tự ý thay preprocessing, labels, folds, metrics, seed policy hoặc artifact schema.
- Không thay nhiều nhóm yếu tố trong cùng một candidate nếu sau đó không thể biết yếu tố nào tạo ra khác biệt.
- Trước khi sửa, ghi rõ hành vi hiện tại và mục tiêu của thay đổi.
- Sau khi sửa, ghi rõ file, notebook heading hoặc cell ID, nội dung thay đổi và lý do.
- Mỗi thay đổi phải liên kết được với candidate chịu ảnh hưởng và kết quả kiểm tra tương ứng.
- Nếu AI tạo thay đổi ngoài phạm vi, phải hoàn tác thay đổi đó trước khi chạy thí nghiệm.

Mỗi bạn phải cập nhật `OPTIMIZATION_CHANGE_LOG.md` trong thư mục của mình. Không chấp nhận mô tả chung như "optimized model" hoặc "improved training".

Đối với notebook, ưu tiên ghi heading và cell ID thay vì line number vì line number trong file `.ipynb` không ổn định.

## 9. Điều kiện để một kết quả được chấp nhận

Một candidate chỉ hợp lệ khi có đủ:

- Resolved configuration.
- Run manifest.
- Training history.
- Latest checkpoint và best checkpoint có identity nhất quán.
- Row-level development predictions.
- Fold-level metrics.
- Independently recomputed global metrics.
- Không có NaN hoặc Inf.
- Không có official-test access trong execution path.
- Có change log khớp với source và candidate matrix.

Không kết luận "improved" nếu mean trajectory NRMSE không thấp hơn mốc so sánh tương ứng theo đúng giao thức trên.

## 10. Kết luận cuối vòng

Báo cáo cuối phải trả lời rõ:

1. Candidate nào thắng theo tiêu chí đã khóa?
2. Candidate đó có vượt `OPT2_06 = 0.28789` hay không?
3. Multimodal có vượt Vibration-only trong cùng điều kiện hay không?
4. Mức cải thiện có ổn định trên 6 folds hay chỉ đến từ một vài bearings?
5. Kết quả có đủ điều kiện để freeze configuration trước official evaluation hay chưa?

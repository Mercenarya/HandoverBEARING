# NHIỆM VỤ 2: TỐI ƯU MULTIMODAL

## 1. Việc cần làm

Phát triển tiếp mô hình `CNNBiLSTMAttentionRUL` từ `OPT2_09`, tập trung vào cách sử dụng vibration, temperature, masks, fusion và attention.

Mục tiêu không phải chỉ làm mô hình phức tạp hơn. Mục tiêu là kiểm tra bằng development evidence xem Multimodal có cải thiện so với Vibration-only hay không.

Đọc `OPTIMIZATION_CONTEXT.md` trước khi sửa notebook hoặc chạy thí nghiệm.

## 2. Hai mốc phải so sánh

### Mốc của chính hướng Multimodal

| Chỉ số | `OPT2_09` |
|---|---:|
| Mean trajectory NRMSE | `0.34792` |
| Mean trajectory NMAE | `0.29597` |
| NRMSE sample standard deviation | `0.10429` |
| Global Window MAE | `4936.6 s` |
| Global Window RMSE | `6714.7 s` |

### Mốc chung cần vượt

`OPT2_06` Vibration-only có mean trajectory NRMSE `0.28789`.

Một Multimodal candidate có thể cải thiện so với `OPT2_09` nhưng vẫn chưa phải mô hình tốt nhất nếu chưa vượt `OPT2_06`.

## 3. Phạm vi được tối ưu

Có thể khảo sát có kiểm soát:

- Mask-aware temperature encoding.
- Fusion gate giữa vibration và temperature.
- Late fusion hoặc gated fusion.
- Modality dropout chỉ trong training.
- Attention configuration.
- Learning rate, scheduler và regularization.
- Cách xử lý fully missing temperature mà vẫn dùng được vibration branch.

Các nguyên tắc không được vi phạm:

- Missing temperature không phải measured zero.
- Temperature mask phải được sử dụng trong model path.
- Bearing không có temperature vẫn phải inference được bằng vibration.
- Không thay preprocessing, labels, folds hoặc scaler provenance.
- Không thêm handcrafted features vào main model, trừ khi đó là hybrid ablation được ghi nhãn rõ và không dùng để thay đổi định nghĩa main model.
- Không tuyên bố temperature hoặc attention có tác dụng nhân quả nếu không có controlled ablation hỗ trợ.

## 4. Thiết kế 6 candidates

Tạo tối đa 6 cấu hình với run ID:

- `OPT3M_01`
- `OPT3M_02`
- `OPT3M_03`
- `OPT3M_04`
- `OPT3M_05`
- `OPT3M_06`

Yêu cầu:

1. `OPT3M_01` phải là reproduction control của `OPT2_09` trong code path mới.
2. Các candidates còn lại phải đại diện cho giả thuyết riêng, dễ giải thích.
3. Mỗi candidate chỉ thay đổi một nhóm yếu tố có liên quan.
4. Candidate matrix phải được khóa trước khi chạy.
5. Không thêm candidates sau khi nhìn thấy kết quả.

Ưu tiên thiết kế giúp trả lời được:

- Mask-aware fusion có tốt hơn fusion hiện tại không?
- Modality dropout có giúp khi temperature bị thiếu không?
- Attention có tạo lợi ích ổn định trên nhiều folds không?
- Multimodal có vượt Vibration-only trong cùng giao thức không?

## 5. Quy trình thực hiện

### Bước A: Kiểm tra đầu vào

- Xác nhận data gate vẫn PASS.
- Kiểm tra vibration, temperature và masks đúng tensor contract.
- Kiểm tra mọi tensor sau missing-value handling đều finite.
- Xác nhận official evaluation đang tắt.

### Bước B: Unit và smoke checks

- Shape check với output `[B, 1]`.
- Gradient check.
- Overfit-one-batch check.
- Fully missing temperature check.
- Repeated inference check khi temperature mask bằng zero.

### Bước C: Screening

- Chạy tối đa 6 candidates trên folds `0` và `2`.
- Dùng cùng seed policy và training budget với hướng Vibration-only.
- Chọn theo trajectory-based rule đã khóa.

### Bước D: Đề cử finalist

- Đề cử tối đa 2 candidates tốt nhất của hướng này.
- Không tự động chạy 6 folds nếu người phụ trách chung chưa phân bổ trong tổng giới hạn 3 finalists.

### Bước E: Confirmation và subgroup analysis

- Candidate được duyệt phải chạy đủ 6 development folds.
- Export row-level predictions.
- Báo cáo toàn bộ development set.
- Báo cáo riêng bearings có usable temperature và fully missing temperature khi dữ liệu cho phép.
- Không diễn giải subgroup nhỏ thành quan hệ nhân quả.

### Bước F: Kiểm tra độc lập

- Notebook `04` tính lại metrics trực tiếp từ prediction CSV.
- Kiểm tra cả global metrics và fold variability.
- So sánh với `OPT2_09`, `OPT2_06` và finalist mới của hướng Vibration-only.

## 6. Notebook và quyền sở hữu

- Chỉ làm việc trong thư mục riêng đã được giao cho bạn.
- Triển khai hướng Multimodal trong `03_deep_learning_training.ipynb` của thư mục này.
- Không sử dụng file hoặc artifact đang được bạn phụ trách hướng Vibration-only tạo ra.
- Notebook markdown, headings, logs, chart labels và table headers phải là English.
- Mỗi code cell chỉ làm đúng một nhiệm vụ và phải có English markdown heading ngay phía trên.
- Không chuyển logic sang `.py`.

## 7. Kiểm soát thay đổi code

- Đây là tối ưu cục bộ từ `OPT2_09`, không viết lại training pipeline.
- Chỉ sửa các cells cần thiết cho giả thuyết Multimodal đã ghi trong candidate matrix.
- Tách thay đổi về mask, fusion, modality dropout và attention thành các candidates có thể so sánh.
- Không thay đồng thời nhiều thành phần nếu sau đó không thể xác định nguyên nhân khác biệt.
- Không dọn dẹp hoặc refactor code không liên quan trong cùng lần sửa.
- Sau mỗi lần dùng AI sửa code, cập nhật ngay `OPTIMIZATION_CHANGE_LOG.md`.
- Ghi notebook heading hoặc cell ID, hành vi trước, hành vi sau, lý do và candidates bị ảnh hưởng.
- Chạy shape, gradient và missing-temperature checks ngay sau thay đổi.
- Nếu source diff có phần ngoài phạm vi, phải loại bỏ phần đó trước khi báo cáo.

## 8. Đầu ra bắt buộc

Phải bàn giao:

- Bản notebook đã thực thi của hướng Multimodal.
- Candidate matrix đã khóa.
- Screening summary của tối đa 6 candidates.
- Shape, gradient, overfit-one-batch và missing-temperature checks.
- Checkpoints và histories có thể resume đúng identity.
- Run manifests.
- Row-level development prediction CSV.
- Fold, global và subgroup metrics.
- Independently recomputed metric report.
- Một report PASS/FAIL về official-test isolation.
- Một kết luận ngắn: vượt `OPT2_09` chưa và vượt `OPT2_06` chưa.
- `OPTIMIZATION_CHANGE_LOG.md` đã điền đầy đủ và khớp với source thực tế.

Tên artifact phải dùng namespace `OPT3M`; không ghi đè OPT2.

## 9. Tiêu chí hoàn thành

Nhiệm vụ chỉ hoàn thành khi:

- Control `OPT3M_01` chạy hợp lệ và đối chiếu được với `OPT2_09`.
- Fully missing temperature vẫn cho prediction finite.
- Tất cả candidates đều được báo cáo trung thực.
- Candidate được confirmation có đủ 6 folds.
- Metrics truy ngược được đến predictions và checkpoints.
- Notebook `04` tái tính metrics thành công.
- Không có official-test access.
- Mỗi thay đổi code đều có lý do và bằng chứng kiểm tra trong change log.

## 10. Nội dung cần báo cáo lại

- Files đã thay đổi.
- Cells hoặc sections đã thêm/sửa.
- Candidate matrix đã chạy.
- Commands hoặc notebook runs đã thực hiện.
- Metrics thực tế quan sát được.
- Kết quả missing-temperature checks.
- Candidate được đề cử và lý do theo selection rule.
- Blockers, assumptions và các run thất bại.
- Danh sách chính xác những cells đã thay đổi và lý do.


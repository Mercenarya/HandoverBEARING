# NHIỆM VỤ 1: TỐI ƯU VIBRATION-ONLY

## 1. Việc cần làm

Phát triển tiếp từ cấu hình `OPT2_06` để tìm một Vibration-only candidate có trajectory prediction tốt hơn và ổn định hơn.

Đọc `OPTIMIZATION_CONTEXT.md` trước khi sửa notebook hoặc chạy thí nghiệm.

## 2. Baseline bắt buộc phải tái sử dụng

| Chỉ số | `OPT2_06` |
|---|---:|
| Mean trajectory NRMSE | `0.28789` |
| Mean trajectory NMAE | `0.22602` |
| NRMSE sample standard deviation | `0.09767` |
| Global Window MAE | `4279.7 s` |
| Global Window RMSE | `6486.8 s` |

Không được gọi candidate mới là tốt hơn nếu chưa vượt baseline theo selection rule chung.

## 3. Phạm vi được tối ưu

Chỉ tập trung vào nhánh vibration và phần temporal regression phía sau nhánh này. Có thể khảo sát có kiểm soát:

- Learning rate và scheduler.
- Temporal hidden dimension.
- Huber delta hoặc loss đã được phê duyệt cho regression.
- Weight decay và dropout.
- Gradient clipping.
- Early-stopping patience trong cùng maximum epoch budget.
- Cách tổng hợp temporal representation nếu vẫn giữ đúng output `[B, 1]`.

Không thay đổi preprocessing, label, split membership hoặc scaler provenance.

Không đưa handcrafted features vào Vibration-only Deep Learning model trong nhiệm vụ này.

## 4. Thiết kế 6 candidates

Tạo tối đa 6 cấu hình với run ID:

- `OPT3V_01`
- `OPT3V_02`
- `OPT3V_03`
- `OPT3V_04`
- `OPT3V_05`
- `OPT3V_06`

Yêu cầu:

1. `OPT3V_01` phải là reproduction control của `OPT2_06` trong code path mới.
2. Mỗi candidate còn lại phải có giả thuyết rõ ràng và chỉ thay đổi một nhóm yếu tố có liên quan.
3. Lập bảng cấu hình đầy đủ trước khi chạy candidate đầu tiên.
4. Không sửa bảng cấu hình sau khi nhìn thấy metrics.

## 5. Quy trình thực hiện

### Bước A: Kiểm tra đầu vào

- Xác nhận data gate vẫn PASS.
- Xác nhận real batch đúng tensor contract.
- Xác nhận cache, split và scaler identity không đổi.
- Xác nhận official evaluation đang tắt.

### Bước B: Screening

- Chạy tối đa 6 candidates trên folds `0` và `2`.
- Dùng cùng seed policy và training budget cho tất cả candidates.
- Xếp hạng bằng selection rule trong `OPTIMIZATION_CONTEXT.md`.

### Bước C: Đề cử finalist

- Đề cử tối đa 2 candidates tốt nhất của hướng này.
- Không tự động chạy 6 folds nếu người phụ trách chung chưa phân bổ trong tổng giới hạn 3 finalists của cả hai hướng.

### Bước D: Confirmation

- Candidate được duyệt phải chạy đủ 6 development folds.
- Export row-level predictions cho từng fold.
- Tính mean, sample standard deviation và global metrics đúng quy tắc.

### Bước E: Kiểm tra độc lập

- Notebook `04` phải đọc prediction CSV và tính lại metrics.
- So sánh metrics tính lại với selection report.
- Sai lệch vượt tolerance phải làm candidate FAIL, không sửa số trong báo cáo bằng tay.

## 6. Notebook và quyền sở hữu

- Chỉ làm việc trong thư mục riêng đã được giao cho bạn.
- Triển khai hướng Vibration-only trong `03_deep_learning_training.ipynb` của thư mục này.
- Không sử dụng file hoặc artifact đang được bạn phụ trách hướng Multimodal tạo ra.
- Notebook markdown, headings, logs, chart labels và table headers phải là English.
- Mỗi code cell chỉ làm đúng một nhiệm vụ và phải có English markdown heading ngay phía trên.
- Không chuyển logic sang `.py`.

## 7. Kiểm soát thay đổi code

- Đây là tối ưu cục bộ từ `OPT2_06`, không viết lại training pipeline.
- Chỉ sửa các cells cần thiết cho candidate matrix đã khóa.
- Không thay đổi nhiều nhóm yếu tố trong một candidate nếu không có controlled ablation.
- Không dọn dẹp hoặc refactor code không liên quan trong cùng lần sửa.
- Sau mỗi lần dùng AI sửa code, cập nhật ngay `OPTIMIZATION_CHANGE_LOG.md`.
- Ghi notebook heading hoặc cell ID, hành vi trước, hành vi sau, lý do và candidates bị ảnh hưởng.
- Chạy smoke check ngay sau thay đổi trước khi chạy screening dài.
- Nếu source diff có phần ngoài phạm vi, phải loại bỏ phần đó trước khi báo cáo.

## 8. Đầu ra bắt buộc

Phải bàn giao:

- Bản notebook đã thực thi của hướng Vibration-only.
- Candidate matrix đã khóa trước khi chạy.
- Screening summary của tối đa 6 candidates.
- Checkpoints và histories có thể resume đúng identity.
- Run manifests.
- Row-level development prediction CSV.
- Fold metrics và independently recomputed metrics.
- Một report PASS/FAIL về official-test isolation.
- Một kết luận ngắn: vượt hay không vượt `OPT2_06`.
- `OPTIMIZATION_CHANGE_LOG.md` đã điền đầy đủ và khớp với source thực tế.

Tên artifact phải dùng namespace `OPT3V`; không ghi đè OPT2.

## 9. Tiêu chí hoàn thành

Nhiệm vụ chỉ hoàn thành khi:

- Control `OPT3V_01` chạy hợp lệ và có thể đối chiếu với `OPT2_06`.
- Tất cả candidate được báo cáo đúng, kể cả candidate kém hơn.
- Candidate được confirmation có đủ 6 folds.
- Metrics có thể truy ngược đến prediction rows và checkpoint.
- Notebook `04` tái tính metrics thành công.
- Không có official-test access.
- Mỗi thay đổi code đều có lý do và bằng chứng kiểm tra trong change log.

## 10. Nội dung cần báo cáo lại

- Files đã thay đổi.
- Cells hoặc sections đã thêm/sửa.
- Candidate matrix đã chạy.
- Commands hoặc notebook runs đã thực hiện.
- Metrics thực tế quan sát được.
- Candidate được đề cử và lý do theo selection rule.
- Blockers, assumptions và các run thất bại.
- Danh sách chính xác những cells đã thay đổi và lý do.


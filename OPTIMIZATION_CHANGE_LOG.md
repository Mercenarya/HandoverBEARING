# OPTIMIZATION CHANGE LOG

File này dùng để ghi lại chính xác các thay đổi code trong vòng tối ưu. Cập nhật ngay sau mỗi lần sửa, trước khi chạy thí nghiệm dài.

## Thông tin chung

- Hướng tối ưu: `<Vibration-only hoặc Multimodal>`
- Người thực hiện: `<Tên>`
- Baseline: `<OPT2_06 hoặc OPT2_09>`
- Namespace mới: `<OPT3V hoặc OPT3M>`
- Ngày bắt đầu: `<YYYY-MM-DD>`

## Candidate matrix đã khóa

| Candidate | Giả thuyết | Thay đổi có chủ đích | Các yếu tố giữ nguyên |
|---|---|---|---|
| `<Run ID>` | `<Câu hỏi cần kiểm chứng>` | `<Thành phần được thay đổi>` | `<Preprocessing, split, seed, budget...>` |

Không sửa bảng này sau khi đã xem kết quả screening. Nếu bắt buộc phải sửa vì lỗi code, ghi rõ sự cố trong phần Deviations.

## Danh sách thay đổi code

Mỗi thay đổi dùng một mục riêng.

### Change `<Số thứ tự>`

- Thời gian: `<YYYY-MM-DD HH:MM>`
- File: `<Tên file>`
- Notebook heading: `<Heading ngay phía trên code cell>`
- Cell ID: `<Cell ID nếu có>`
- Candidates bị ảnh hưởng: `<Run IDs>`
- Hành vi trước khi sửa: `<Code đang làm gì>`
- Hành vi sau khi sửa: `<Code sẽ làm gì>`
- Lý do: `<Giả thuyết hoặc lỗi cần sửa>`
- Phạm vi thay đổi: `<Các functions, classes hoặc config keys>`
- Phần được giữ nguyên: `<Data, split, metric, output schema...>`
- Kiểm tra đã chạy: `<Shape, gradient, smoke, one-batch...>`
- Kết quả kiểm tra: `<PASS/FAIL và evidence path>`

## Kết quả theo candidate

| Candidate | Screening folds | Mean trajectory NRMSE | Mean trajectory NMAE | Global Window RMSE | Trạng thái | Evidence |
|---|---|---:|---:|---:|---|---|
| `<Run ID>` | `<Folds>` | `<Value>` | `<Value>` | `<Seconds>` | `<PASS/FAIL>` | `<Relative path>` |

Chỉ điền metrics được tạo từ notebook đã thực thi. Không chép số từ màn hình hoặc nhập số ước lượng.

## Deviations và lỗi phát sinh

Ghi mọi khác biệt so với candidate matrix ban đầu:

| Thời gian | Candidate | Sự cố | Cách xử lý | Có thay đổi giao thức không? |
|---|---|---|---|---|
| `<Time>` | `<Run ID>` | `<Mô tả>` | `<Mô tả>` | `<Có/Không>` |

Không âm thầm thay candidate, seed, fold, epoch budget hoặc metric khi run gặp lỗi.

## Tóm tắt cuối cùng

- Các files đã thay đổi: `<Danh sách>`
- Các notebook cells đã thay đổi: `<Headings hoặc cell IDs>`
- Tổng số dòng logic thực sự thay đổi: `<Số lượng gần đúng, không tính notebook outputs>`
- Candidate tốt nhất: `<Run ID>`
- Baseline so sánh: `<Run ID và metric>`
- Mức thay đổi mean trajectory NRMSE: `<Giá trị và phần trăm>`
- Kết luận: `<Tốt hơn, không tốt hơn hoặc chưa đủ bằng chứng>`
- Thay đổi ngoài phạm vi còn tồn tại: `<Không hoặc mô tả rõ>`


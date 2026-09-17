# Báo cáo Day 5 — Segmentation Data

Báo cáo này tổng hợp các export CVAT hiện có trong `submissions/` và kết quả tự kiểm local ngày 18/09/2026.

- Mã học viên theo lớp: 2A202602287
- Ngày / CVAT local: 17/09/2026 / CVAT local
- Công cụ đã dùng: công cụ mask trong CVAT; script kiểm cấu trúc và scorer local bằng Python 3.11

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: theo thứ tự annotation còn lưu trong export, object đầu tiên là người phụ nữ mặc áo dài trắng ở giữa ảnh `000000181542.jpg` (annotation ID 1). Export không lưu nhật ký thao tác chi tiết hơn nên đây là bằng chứng gần nhất về thứ tự vẽ.
- Class và quy tắc tôi dùng để chọn biên: class `person`; bám theo phần cơ thể và quần áo còn nhìn thấy, dừng tại mép thật của người hoặc vị trí bị xe máy che, không tự đoán phần khuất phía sau.
- Thông tin về gợi ý sau đó: export không lưu thông tin công cụ tạo từng mask; trong vòng kiểm/sửa bằng scorer tôi không lấy mask từ ground truth hay sửa trực tiếp JSON/RLE trong ZIP, mà giữ quy trình sửa trong CVAT rồi export lại.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `hard_panoptic`, các xe `car` nhỏ trong ảnh `000000350023.jpg`.
- Lỗi thuộc loại: thiếu-thừa vật và gộp-tách; có nhiều mask `car` thừa hoặc phân mảnh không ghép được với một xe thật.
- Bằng chứng tôi nhìn thấy: lần chấm trước có `car` TP/FP/FN = 25/18/1; sau khi rà lại còn 25/4/1. Số TP và FN giữ nguyên trong khi FP giảm mạnh, cho thấy các mask thừa đã được loại bỏ mà không làm mất xe đã nhận đúng.
- Quy tắc và hành động sửa: mỗi xe là một instance; xóa mask trùng/mảnh nhỏ không đại diện cho xe riêng, giữ các xe có đường bao và vị trí độc lập, không gộp xe đứng sát nhau.
- Sau sửa đã Save và export lại chưa? Đã Save trong CVAT, export lại `hard_panoptic.zip` và chạy scorer local.

Kết quả liên quan sau khi sửa: PQ của class `car` tăng từ `0.593` lên `0.744`; điểm `hard_panoptic` tăng từ `18.2/30` lên `19.5/30`, và tổng ba tier tăng từ `49.9/82` lên `51.2/82`. Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Tôi không tự ghi PASS/top 3/bonus và không đưa file ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `000000181542.jpg`, người phụ nữ áo dài trắng ở giữa ảnh bị các xe máy đi ngang che một phần | Vẽ cả phần cơ thể suy đoán phía sau xe, hoặc chỉ vẽ phần còn nhìn thấy | Quy tắc của task là chỉ gán phần nhìn thấy, không tự đoán biên sau vật che | Chọn class `person` và chỉ giữ vùng cơ thể/quần áo quan sát được |
| `000000017627.jpg` (`cp2_slice`), dãy ô tô sát nhau ở giữa ảnh | Gộp các xe cùng class thành một mask, hoặc tách từng xe thành instance riêng | Vẫn thấy đường bao, khe và vị trí độc lập của từng xe; instance segmentation cần định danh từng vật | Tách mỗi ô tô thành một instance `car` riêng |
| `7d83710e-4697c3b2.jpg` (`cp4_curb`), ranh giữa mặt đường tối bên trái và nền bê tông nâng cao bên phải | Xem vùng sát bó vỉa là `road`, hoặc xem phần nền nâng cao là `sidewalk` | Dựa vào bó vỉa, cao độ và chức năng đi bộ thay vì chỉ dựa vào màu vật liệu | Chọn mặt xe chạy là `road`, phần bê tông nâng cao phía phải là `sidewalk`, lấy bó vỉa làm ranh |

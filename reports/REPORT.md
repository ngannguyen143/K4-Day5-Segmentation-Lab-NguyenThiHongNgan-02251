# Báo cáo Day 5 — Segmentation

- Mã học viên theo lớp: 2A202602251
- Ngày / CVAT local: 17/09/2026 / CVAT local của lớp
- Công cụ đã dùng: Brush / Polygon

Tôi đã kiểm tra và Save annotation trên CVAT trước khi export. Không dùng SAM hoặc gợi ý tự động.

## 1. Bài đã nộp

Các ZIP dưới đây đã được export và đặt trong `submissions/`. Cột điểm chỉ là điểm tối đa theo rubric, không phải điểm tự chấm.

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

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, object `car` đầu tiên trong danh sách Objects.
- Class và quy tắc tôi dùng để chọn biên: `car`; chỉ vẽ phần thân xe nhìn thấy, bám theo biên xe và không đoán phần bị che.
- Gợi ý tự động: không dùng; tôi tiếp tục kiểm từng mask bằng Brush/Polygon.
- Kiểm tra trước khi Save: class, biên mask và số object trong Objects.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp2_slice` / `000000017627.jpg` / vùng hai xe cùng lớp ở gần nhau.
- Lỗi thuộc loại: gộp-tách.
- Bằng chứng tôi nhìn thấy: khe giữa hai xe cho thấy đây là hai vật riêng, không phải một mask duy nhất.
- Quy tắc và hành động sửa: tách hai xe thành hai instance riêng, kiểm tra lại biên từng mask rồi export lại `cp2_slice.zip`.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại.

Tôi chưa ghi điểm vì chưa có reference/điểm chính thức. Ground truth không được đưa vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` / `000000144300.jpg` / kính hoặc khe trong vật | Khoét thành lỗ hoặc giữ trong mask vật | Quy tắc task yêu cầu kính/khe ở trong mask; không khoét tùy tiện | Giữ trong mask vật. |
| `cp4_curb` / `7d83710e-4697c3b2` / ranh bó vỉa | Road hoặc sidewalk khi màu mặt đường gần giống nhau | Xác định theo chức năng và bó vỉa, không chỉ theo màu | Chọn theo phía sidewalk/road của bó vỉa. |
| `cp3_thin` / `839f7736-abe28069` / pole hoặc traffic sign mảnh | Bỏ qua vì quá nhỏ hoặc giữ đúng class | Phóng to, dùng brush 2–3 px và kiểm tên class của task | Giữ phần nhìn thấy với class phù hợp; không tô lan sang sky/road. |
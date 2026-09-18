# Phiếu tra nhanh gán nhãn Day 5

## 1. Chọn đúng loại trước khi vẽ

| Loại | Câu hỏi | Task |
| --- | --- | --- |
| **Semantic** | Pixel này thuộc **loại vùng** nào? Mỗi lớp là một vùng, không đếm từng vật. | `easy_semantic`, `cp3_thin`, `cp4_curb`, `cp6_coverage` |
| **Instance** | Pixel này thuộc **vật nào**? Mỗi vật đếm được là một object/mask riêng. | `medium_instance`, `cp1_holes`, `cp2_slice`, `cp5_occlusion` |
| **Panoptic** | Vùng thuộc lớp nào **và** vật đếm được nào? | `hard_panoptic` |

Mask là vùng pixel, không phải bounding box. **Stuff** là vùng không đếm từng cá thể (`road`, `sky`...); **thing** là vật đếm được (`car`, `person`...). Tên lớp phải khớp từng ký tự trong `classes.json` của **chính task đó**: `traffic sign` khác `traffic_sign`. Không dùng chung danh sách lớp giữa các task.

## 2. Lớp chuẩn theo từng task

| Task | Loại | Tên lớp phải dùng |
| --- | --- | --- |
| `easy_semantic` | semantic | `road`, `sidewalk`, `building`, `vegetation`, `sky` |
| `medium_instance` | instance | `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck` |
| `hard_panoptic` | panoptic | Stuff: `road`, `sidewalk`, `building`, `vegetation`, `sky`; Thing: `person`, `car`, `bus`, `truck`, `motorcycle`, `bicycle`, `traffic light` |
| `cp1_holes` | instance | `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck` |
| `cp2_slice` | instance | `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck` |
| `cp5_occlusion` | instance | `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck` |
| `cp3_thin` | semantic | `pole`, `traffic sign`, `sky`, `road` |
| `cp4_curb` | semantic | `road`, `sidewalk` |
| `cp6_coverage` | semantic | `road`, `sidewalk`, `building`, `vegetation`, `sky`, `car`, `person` |

`classes.json` là nguồn kiểm tra tên lớp/metadata. Nếu dùng **Labels → Raw** của CVAT, dán toàn bộ `cvat-labels.json` cùng thư mục task; không dán `classes.json` vào Raw. Nếu thêm bằng **Constructor**, thêm từng tên trong bảng trên.

## 3. Quy tắc hình học

- Vẽ sát **phần nhìn thấy**; không tự đoán biên phía sau vật bị che.
- Hai vật cùng lớp dù sát nhau vẫn là **hai instance**.
- Một vật bị vật khác che có thể có các mảng nhìn thấy rời nhau nhưng vẫn là **một instance**; không tách chỉ vì bị che.
- Kính, khe và chi tiết bên trong vật **không tự động là lỗ**. Với `cp1_holes`, giữ chúng trong mask theo quy tắc task, không khoét tùy tiện.
- Ranh `road`–`sidewalk` dựa vào chức năng và bó vỉa, không chỉ dựa vào màu; áp dụng đặc biệt cho `cp4_curb`.
- Với `cp3_thin`, phóng to và dùng brush khoảng **2–3 px** để giữ pole/traffic sign, tránh brush quá dày.
- Với `cp6_coverage`, rà toàn ảnh để không bỏ sót vùng nhìn thấy thuộc một trong 7 lớp; không tô bừa vùng không chắc.
- Với panoptic, vẽ cả stuff và **từng thing**; `car` chung không thay cho `car #1`, `car #2`. Kiểm vùng chồng lấn và khe trống bằng mắt.

## 4. Quy trình CVAT và export

1. Tạo **một task cho mỗi mã** trong bảng, đặt tên đúng mã task và chỉ tải ảnh trong thư mục `data/.../<task>/images/`.
2. Tạo labels từ `cvat-labels.json` đúng task hoặc thêm từng tên từ `classes.json`. Nếu task đã có annotation, không dán đè Raw.
3. Mở Job, chọn Brush/Polygon, vẽ phần nhìn thấy; kiểm class và số object trong **Objects**. SAM/Intelligent Scissors chỉ là tùy chọn.
4. Bấm **Save**, đổi ảnh, quay lại kiểm một ảnh để chắc annotation còn đó.
5. Chọn Job → **Export job dataset** theo bảng dưới. Giữ nguyên nội dung ZIP và đổi tên ZIP bên ngoài thành mã task.

| Task type | Format CVAT | Tên ZIP trong `submissions/` |
| --- | --- | --- |
| Semantic: `easy_semantic`, `cp3_thin`, `cp4_curb`, `cp6_coverage` | `Segmentation mask 1.1` | `<mã_task>.zip` |
| Instance: `medium_instance`, `cp1_holes`, `cp2_slice`, `cp5_occlusion` | `COCO 1.0` | `<mã_task>.zip` |
| Panoptic: `hard_panoptic` | `COCO 1.0` | `hard_panoptic.zip` |

Không đổi sang format khác và không sửa JSON/PNG trong ZIP bằng tay. Nếu format không có hoặc export lỗi: giữ dữ liệu đã Save, ghi task/thời điểm/lỗi vào `REPORT.md` và báo coach.

## 5. Tự QC trước khi nộp

Kiểm theo đúng thứ tự:

1. Đúng task, đúng đủ ảnh và đúng loại semantic/instance/panoptic.
2. Tên class khớp `classes.json`; không có class lạ.
3. Đủ vùng/vật; không bỏ sót, tô thừa, gộp hai vật hoặc tách sai một vật.
4. Biên bám phần nhìn thấy; không ăn nền/bóng; kiểm ranh road–sidewalk, lỗ và nét mảnh.
5. Với panoptic, kiểm stuff/thing, chồng lấn và khoảng trống.
6. Đã **Save**, export đúng format, ZIP đúng mã task và đúng ảnh.

Có thể chạy kiểm cấu trúc từ thư mục gốc (không cần reference):

```bash
python3 scripts/inspect_submissions.py --dir submissions
```

Trên Windows có thể thay `python3` bằng `py -3`. `OK` chỉ xác nhận cấu trúc, ảnh, class và dạng mask; không chứng minh mask đúng. `THIẾU` là chưa có ZIP; `LỖI` cần sửa trong CVAT rồi Save/export lại. Với COCO, số `annotations` là số mask đã nộp, không phải số object đúng.

## 6. Report và nộp bài

Điền `REPORT.md` ở gốc fork, gồm: task/ảnh đã làm và ZIP tương ứng; object Medium đầu tiên tự vẽ trước gợi ý; một lỗi thật đã sửa; ba ca chưa chắc hoặc đã cân nhắc. Không tự điền điểm, PASS, bonus hay top 3.

Đưa các ZIP đã làm vào `submissions/`, push/upload `REPORT.md` và ZIP lên **fork cá nhân**, rồi nộp link fork trên VLearn trong vòng **24 giờ sau buổi lab**. Không tạo ZIP rỗng cho task chưa làm. Ground truth/reference không có trong repo học viên và không được đưa vào fork công khai.

Tự chấm Easy/Medium/Hard chỉ dùng được sau khi coach phát reference; GitHub Actions hoặc `scoring/` khi đó cho phản hồi tối đa **82 điểm** (20 + 32 + 30), không thay rubric 100 điểm và không tự quyết định PASS/bonus. Khi chưa có reference, chỉ tự QC bằng mắt và kiểm cấu trúc ZIP.

## Khi không chắc

Ghi trong `REPORT.md`: tên ảnh, vị trí, hai cách hiểu, dấu hiệu nhìn thấy, quy tắc đã áp dụng và quyết định/câu hỏi cho coach. Không ép đoán để đạt coverage. Không vào được CVAT thì lưu ảnh lỗi và thời điểm; không tự cài một stack CVAT khác giữa giờ lab.


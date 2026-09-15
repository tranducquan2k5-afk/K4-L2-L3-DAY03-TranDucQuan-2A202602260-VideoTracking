# Peer review — Day 3

Chép file này thành 
eports/review_partner.md. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Trần Đức Quân |
| Reviewer | Nguyễn Văn A |
| Pair ID | K4-P03 |
| CVAT version | CVAT Online (app.cvat.ai v2.10.0) |
| Thời điểm review | 15/09/2026 16:30 |

## Danh sách finding

Mỗi finding bắt buộc có rame + ID + lỗi + sửa thế nào. Nếu chưa thống nhất,
dùng 
eeds-review; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 148-150 | 149-151 | 6 | Bbox treo / Bbox thừa | Xe lớn ID 6 đã rời mép trái ảnh ở frame 148, nhưng tác giả vẫn giữ 3 bbox dính mép trái x=0 (dính vệt bóng đổ). Vi phạm Exit Rule. | Bấm outside (phím O) dứt khoát tại frame 148 ngay khi mép thân xe vừa qua khỏi khung hình. | fixed |
| 2 | 58-77 | 59-78 | 7 | Bbox sớm / Chưa đạt ngưỡng | Xe ô tô ở làn xa mới là vệt mờ dưới 10px, bị xe ID 6 che khuất, chưa xác định chắc chắn 4 bánh. Vi phạm Entry Threshold (Quy tắc 15-pixel). | Bắt đầu track từ frame 79 khi xe lộ diện rõ và kích thước đạt > 15px. | fixed |
| 3 | 0-88 | 1-89 | 3 | Gán nhầm class | Track ID 3 là xe máy chở thùng hàng cồng kềnh di chuyển từ giữa ảnh sang trái. Không thuộc schema vehicle (chỉ gán xe 4 bánh). | Xóa toàn bộ track ID 3 để tránh False Positive. | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đã có 8 track xe 4 bánh hợp lệ sau khi loại bỏ 2 track xe máy |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Toàn bộ các track duy trì ID duy nhất, IDSW = 0 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 7 bị Track 6 che ở frame 80-110 vẫn giữ nguyên ID 7 |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS (sau khi sửa) | Đã bấm outside đúng tại frame 148 (ID 6) và frame 168 (ID 10) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox ôm sát visible contour, không vẽ phần khuất |
| Frame giữa hai keyframe không bị interpolation drift | PASS (sau khi sửa) | Đã bổ sung keyframe giữa ở frame 85, 105 cho Track 7, 8 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Đã kiểm tra bằng check_mot_labels.py không có lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 3/3 findings đã điền cách sửa và đánh dấu fixed |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Toàn bộ timeline liên tục, không nhảy ID hay trùng ID |
| 2 — endpoint/scope | ĐÃ SỬA | Đã cắt tỉa đầu/cuối của ID 6, 7, 8, 10 theo đúng quy chuẩn |
| 3 — geometry/interpolation | PASS | Bbox ôm khít thân xe, IoU đạt chuẩn sau khi nắn keyframe |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Finding #1 (Track 6 dính bóng đổ ngoài rìa ảnh ở frame 149-151) - áp dụng Exit & Shadow Rule: bấm outside ngay khi thân xe thật sự rời khỏi khung nhìn.
2. Một finding tác giả đóng là 
ot-a-defect, kèm lý do (nếu có): Không có (toàn bộ các finding đều được tác giả đồng thuận và đóng fixed).
3. Một rule cần Lab Coach làm rõ (nếu có): Quy chuẩn gán nhãn đối với các phương tiện đặc thù như xe bán tải chở đồ nhô cao hoặc xe kéo rơ-moóc thùng rời khi rơ-moóc bị che khuất.

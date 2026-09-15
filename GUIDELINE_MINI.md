# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Trần Đức Quân`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có):
- Xe ba gác, xe xích lô, xe máy điện/scooter hai bánh: **KHÔNG gán** (chỉ gán phương tiện cơ giới bốn bánh trở lên).
- Xe kéo rơ-moóc / xe đầu kéo container: Gán thành **một bbox duy nhất** bao phủ cả đầu kéo và rơ-moóc thùng hàng phía sau nếu là khối di chuyển liền mạch.
- Xe bị che khuất > 85% hoặc ở quá xa (chưa nhận diện rõ ràng đèn, kính chắn gió hoặc kết cấu bánh xe): **KHÔNG gán** cho đến khi thấy rõ ít nhất 15-20% thân xe và xác định chắc chắn là xe bốn bánh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Thời gian che khuất ngắn (< 2 giây) khi đi qua sau cây cối, cột đèn hoặc bị xe khác vượt mặt; quỹ đạo và vận tốc di chuyển của xe có tính quy luật cao (quán tính chuyển động), đủ độ tin cậy để liên kết cùng một identity mà không sợ nhầm lẫn. |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới với ID mới khi xe xuất hiện trở lại (trừ khi xe dừng đỗ tĩnh trước khi bị che và xuất hiện lại đúng vị trí đó) | Sau hơn 2 giây (> 25 frame), độ bất định về chuyển động rất lớn (xe có thể đã rẽ, quay đầu, đổi tốc độ hoặc bị thay thế bởi một xe khác cùng màu); việc cấp ID mới an toàn hơn là gán nhầm identity (tránh false association / ID switch). |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** (cấp ID mới) | Không có cơ sở quan sát liên tục ngoài vùng nhìn của camera để khẳng định danh tính; tuân thủ đúng chuẩn MOT benchmark (rời khỏi khung hình = kết thúc trajectory). |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID của từng xe trước và sau giao cắt. Xe ở gần/layer trên giữ bbox bao trọn thân xe nhìn thấy; xe ở xa/layer dưới co hẹp bbox theo phần còn nhìn thấy. Nếu xe ở xa bị che hoàn toàn dưới 25 frame thì ngắt bbox rồi khôi phục lại đúng ID cũ khi lộ diện. | Dựa vào vector hướng di chuyển (quán tính chuyển động thẳng đều) và đặc trưng hình thái/màu sơn của từng xe để bảo toàn identity xuyên suốt giao cắt, tránh hoán đổi ID (giữ ID switch = 0). |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh (tuân thủ visible bounding box). |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**, không bao gồm phần thân bị che khuất. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **chiều rộng hoặc chiều cao >= 15 pixel** và nhận diện được tối thiểu 2 đặc trưng (kính lái, đèn xe, mặt ca-lăng, bánh xe). Bỏ qua các đốm pixel mờ dưới 15px để tránh tạo False Positive. |
| Xe đang đỗ, không di chuyển | Gán một track liên tục với cùng một ID tĩnh suốt toàn bộ các frame xe xuất hiện. Đặt keyframe ở frame đầu tiên và frame cuối cùng, kiểm tra tọa độ bbox không bị xê dịch. |
| Keyframe đặt dày ở đâu | Đặt dày (cách nhau 2 - 5 frame) tại các đoạn xe tăng/giảm tốc đột ngột, chuyển làn, quay đầu, đổi hướng rẽ, hoặc khi xe tiến rất gần camera làm tỷ lệ kích thước thay đổi nhanh. Ở đoạn chạy thẳng đều có thể giãn cách 15 - 25 frame. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 1 – 89 / ID 3
- Tình huống: Một phương tiện di chuyển ở làn xa từ giữa khung hình (x=553.4) về phía mép trái (x=0.0). Kích thước tương đối nhỏ (w ~ 37px, h ~ 25px). Ban đầu khó phân biệt giữa xe ô tô con cỡ nhỏ (hatchback đi xa) và xe máy chở hàng cồng kềnh.
- Quyết định: Ban đầu nhóm đã gán nhãn là ID 3 thuộc `vehicle`, nhưng sau khi rà soát kỹ và đối chiếu gold reference, nhóm xác định đây là xe máy chở đồ (hai bánh) và loại bỏ khỏi nhãn `vehicle`.
- Lý do: Đối tượng có tiết diện ngang hẹp, không có kết cấu hai trục bánh xe song song của ô tô; xe máy nằm ngoài schema gán nhãn của Ngày 3 (chỉ gán xe bốn bánh).

### Ca 2
- Clip / frame / ID: `clip_01` / frame 59 – 78 / ID 7 (ứng với xe GT track 5)
- Tình huống: Xe ô tô xuất hiện từ rìa phải ở khoảng cách rất xa, kích thước ban đầu chỉ có w ~ 8px, h ~ 27px, bị mờ nhoè do độ phân giải thấp và bị che khuất một phần bởi xe ID 6 đang chạy song song phía trước.
- Quyết định: Ban đầu nhóm gán sớm từ frame 59. Sau đó quyết định lùi thời điểm bắt đầu track về frame 79 (khi xe tách hẳn khỏi xe trước, kích thước đạt > 15px và thấy rõ mặt ca-lăng trước).
- Lý do: Bắt đầu track quá sớm khi xe chưa đạt ngưỡng nhận dạng tối thiểu dẫn đến 20 frame FP (ghost bbox). Phải tuân thủ quy tắc ngưỡng nhận dạng tối thiểu (>= 15px).

### Ca 3
- Clip / frame / ID: `clip_01` / frame 149 – 151 / ID 6 (và frame 169 – 171 / ID 10)
- Tình huống: Xe kích thước lớn ID 6 di chuyển nhanh và lướt ra khỏi mép trái màn hình. Tại frame 148, thân xe đã ra khỏi khung hình 95%, tại frame 149-151 chỉ còn vệt bóng đen mờ của đuôi xe và bóng đổ trên mặt đường sát mép x=0.
- Quyết định: Bấm `outside` (phím `O`) dứt khoát tại frame 148 ngay khi mép thân xe rời khung; xóa bỏ 3 bbox ở frame 149-151.
- Lý do: Bbox chỉ ôm thân xe thực tế, không được ôm vệt bóng đổ (shadow artifact) trên mặt đường ở rìa ảnh để tránh lỗi bbox treo (FP).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy tắc ngưỡng nhận dạng tối thiểu (Entry Threshold - Quy tắc 15 pixel)**: Không được bắt đầu track khi vật thể chỉ là một đốm pixel mờ ở đường chân trời. Chỉ bắt đầu tạo track khi chiều rộng/chiều cao đạt tối thiểu 15 pixel và nhìn thấy rõ ít nhất 2 đặc trưng nhận dạng của xe bốn bánh (đèn, kính chắn gió, cản trước hoặc kết cấu khối xe).
- **Quy tắc kết thúc track và loại trừ bóng đổ (Exit & Shadow Rule)**: Khi xe đi ra khỏi khung hình, phải bấm `outside` ngay tại frame cuối cùng mà phần cứng của thân xe còn chạm mép ảnh. Tuyệt đối không kéo dài bbox bám theo bóng đổ (shadow) hoặc vệt mờ quang học ở rìa ảnh.
- **Quy tắc phân biệt xe máy chở hàng / ba gác**: Khi gặp xe ở xa có hình khối cồng kềnh, phải zoom 200% và tua tới lui 5-10 frame để quan sát tiếp xúc bánh xe với mặt đường; nếu chỉ có một vệt bánh hoặc hai bánh đơn dọc thì kiên quyết không gán.

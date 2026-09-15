# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Trần Đức Quân`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (app.cvat.ai), chế độ Rectangle Track |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `75` phút |
| Số track đã vẽ trong `clip_01` | `10` track |
| Số keyframe trung bình mỗi track | `7.2` keyframes/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đổi kích thước và vận tốc cực nhanh (Track 6)**: Ở các frame đầu (frame 51-80), xe ở xa di chuyển chậm với kích thước nhỏ (w ~ 12-50px), nhưng từ frame 100-148 xe tiến sát camera, kích thước phình to đột biến (w > 470px) và lướt nhanh ra khỏi khung hình trong vài frame. Xử lý: Khi xe ở xa đặt keyframe thưa (15-20 frame), khi xe vào gần đặt keyframe dày (3-5 frame) và căn chỉnh góc bbox bám sát thân xe nhìn thấy; bấm `outside` (phím `O`) ngay frame 148 khi đuôi xe vừa chạm mép ngoài.
2. **Hiện tượng che khuất một phần (Occlusion) và giao cắt giữa hai xe cùng chiều (Track 6 và Track 7)**: Track 7 di chuyển ở làn trong, bị thân xe Track 6 che khuất một phần trong nhiều frame liên tiếp (frame 80-110). Xử lý: Tuân thủ nghiêm ngặt quy tắc visible bbox — chỉ vẽ bbox ôm phần nhìn thấy được của Track 7, không suy đoán phần thân bị xe trước che lấp; giữ nguyên ID của Track 7 xuyên suốt quá trình bị che khuất và khi lộ diện hoàn toàn.
3. **Xe đỗ tĩnh ven đường suốt toàn bộ thời lượng video (Track 2)**: Xe đỗ ở lề đường xuất hiện từ frame 1 đến frame 190. Xử lý: Đặt keyframe ở frame 1 và để CVAT tự động nội suy tĩnh đến frame 190, tuyệt đối không click tạo thêm các keyframe thừa dọc đường đi để tránh rung lắc (jitter/drift) do tay kéo thủ công làm giảm LocA/MOTP.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Quan sát số ID chạy liên tục trên từng xe để rà soát ID switch. Kết quả: Không có hiện tượng nhảy ID hay tráo đổi ID giữa chừng (IDSW = 0); kiểm tra được tính liên tục của xe đỗ tĩnh (ID 2).
- Lượt 2: Kiểm tra frame đầu và frame cuối của từng track. Bắt được lỗi Track 6 và Track 10 bị kéo dài thừa 3 frame ngoài rìa ảnh sau khi thân xe đã ra khỏi khung (do dính bóng đổ); phát hiện Track 7 và Track 8 được bắt đầu hơi sớm khi xe chỉ mới là đốm mờ ở chân trời.
- Lượt 3: Tua vào giữa các khoảng cách keyframe xa nhau (đặc biệt các đoạn xe cua hoặc chuyển làn như Track 7, 8, 10). Bắt được một số frame bị hở nhẹ bbox (interpolation drift, IoU tụt xuống ~0.50 - 0.58); đã bổ sung keyframe chốt ở giữa để tăng độ khít (LocA).

Kiểm chéo với: `Nguyễn Văn A (Pair ID: K4-P03)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3`. Số lỗi bạn ấy tìm được trong bản của bạn: `2`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- Ca phương tiện ở làn xa di chuyển từ giữa ảnh sang trái (Track 3 trong bản pre-gold của tôi, frame 1-89): Bạn cùng nhóm không gán vì xác định đó là xe máy chở đồ cồng kềnh; tôi ban đầu gán vì nhìn từ xa tưởng xe bán tải mini. Khi zoom 200% và đối chiếu gold, xác nhận đây là xe máy 2 bánh.
- Luật còn thiếu trong `GUIDELINE_MINI.md`: Cần bổ sung "Quy tắc 15-pixel" (ngưỡng kích thước tối thiểu để bắt đầu track) và quy tắc kiểm tra vệt tiếp xúc bánh xe/kính chắn gió khi nghi ngờ xe hai bánh chở hàng.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `e5ac089f0a53c344136d51f7d902bae6d57b4242720179caac1e296429cae7d4` |
| Thời điểm khóa | `2026-09-15T10:34:48.060989+00:00` |
| Số row / frame / track trước khi mở reference | `759 row / 190 frame / 10 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.6923 | 0.6168 | 0.7777 | 0.8541 | 0.8438 | 0.6370 | 0.8384 | 197 | 11 | 0 |
| Sau rework | 0.8715 | 0.9420 | 0.8062 | 0.8625 | 0.9680 | 0.9686 | 0.8450 | 7 | 11 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`):
- Bản pre-gold: **chưa** (IDF1=0.844 đạt, MOTP=0.838 đạt, nhưng MOTA=0.637 < 0.75 do FP=197 quá lớn).
- Sau rework: **có** (cả 3 chỉ số đều vượt ngưỡng: IDF1=0.968 >= 0.80, MOTA=0.969 >= 0.75, MOTP=0.845 >= 0.70).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost track (gán nhầm ngoài class) | 1 – 89 | 3 | Xóa toàn bộ track ID 3 (đây là xe máy hai bánh chở đồ cồng kềnh, không thuộc schema vehicle) |
| Ghost track (gán nhầm ngoài class) | 1 – 36 | 5 | Xóa toàn bộ track ID 5 (đối tượng ở mép lề đường, không phải xe bốn bánh) |
| Bbox sớm (chưa đạt entry threshold) | 59 – 78 | 7 | Xóa 20 bbox thừa; bắt đầu track 7 từ frame 79 khi xe lộ diện rõ và kích thước đạt > 15px |
| Bbox sớm (chưa đạt entry threshold) | 79 – 100 | 8 | Xóa 22 bbox thừa; bắt đầu track 8 từ frame 101 khi xe tách khỏi hậu cảnh và đạt ngưỡng nhận dạng |
| Bbox trễ / Treo (quên bấm outside) | 149 – 151 | 6 | Bấm outside (phím O) dứt khoát tại frame 148; xóa 3 bbox thừa dính vệt bóng đổ ở mép trái |
| Bbox sớm (chưa đạt entry threshold) | 51 – 53 | 6 | Xóa 3 bbox ở frame 51-53; bắt đầu track 6 từ frame 54 khi thân xe tiến vào khung hình |
| Bbox sớm (chưa đạt entry threshold) | 103 – 105 | 9 | Xóa 3 bbox ở frame 103-105; bắt đầu track 9 từ frame 106 |
| Bbox trễ / Treo (quên bấm outside) | 169 – 171 | 10 | Bấm outside tại frame 168 khi xe vừa thoát khỏi mép phải ảnh; xóa 3 bbox thừa ngoài rìa |
| Bbox trôi (Loose box drift) | 83-88, 102-112 | 7, 8 | Bổ sung keyframe ở giữa các đoạn chuyển hướng để nắn bbox khít thân xe, nâng IoU từ ~0.51 lên > 0.85 |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt` / ByteTrack (`bytetrack.yaml`) & BoT-SORT + ReID (`/content/Day3-Lab/configs/trackers/botsort-reid.yaml`) |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` (COCO classes: car, bus, truck) |
| device | `0` (CUDA GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold (pre-gold) | 0.6923 | 0.6168 | 0.7777 | 0.8541 | 0.8438 | 0.6370 | 0.8384 | 197 | 11 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.6492 | 0.5529 | 0.7643 | 0.8593 | 0.7917 | 0.6206 | 0.8402 | 82 | 203 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Ở bản pre-gold của tôi: IDF1 (0.8438) cao hơn MOTA (0.6370). Trường hợp của tôi MOTA thấp là do lượng False Positives lớn (FP = 197 do gán nhầm 2 track xe máy ngoài class và vẽ sớm/muộn ở biên).
- Tuy nhiên, xét về bản chất thiết kế metric: Nếu một hệ thống có **MOTA rất cao nhưng IDF1 lại thấp**, điều đó phản ánh hệ thống gặp **lỗi nghiêm trọng về việc duy trì định danh (Identity Association / ID Switches / Track Fragmentation)**, trong khi khâu phát hiện vị trí (Detection: FP và FN) vẫn hoạt động tốt.
- Lý do MOTA không phạt nặng lỗi ID:
  Công thức của MOTA là:
  $$\text{MOTA} = 1 - \frac{\sum (\text{FP}_t + \text{FN}_t + \text{IDSW}_t)}{\sum \text{GT}_t}$$
  Trong đó, mỗi lần xảy ra hoán đổi ID ($\text{IDSW}$), MOTA chỉ cộng thêm đúng 1 đơn vị phạt tại frame xảy ra chuyển đổi, bất kể track đó dài hàng trăm frame sau đó bị gán sai ID. Ví dụ: một xe tồn tại 100 frame, ở frame 50 bị đổi sang ID mới, MOTA chỉ bị phạt đúng 1 điểm trên tổng số 100 GT boxes (tổn thất chỉ 1% điểm MOTA). Ngược lại, IDF1 tính toán dựa trên sự trùng khớp ID toàn cục dọc theo toàn bộ vòng đời ($\frac{2 \cdot \text{IDTP}}{2 \cdot \text{IDTP} + \text{IDFP} + \text{IDFN}}$), nên khi một track 100 frame bị chẻ đôi thành 2 track 50 frame, IDF1 sẽ bị phạt nặng nề (mất tới một nửa số frame định danh đúng, điểm tụt sâu). Do đó, MOTA đo lường nặng về detection coverage ở từng frame, còn IDF1 đo lường sự nhất quán identity xuyên suốt chuỗi thời gian.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- Số liệu so sánh giữa Control (ByteTrack) và Treatment (BoT-SORT + ReID):
  - **IDF1**: BoT-SORT + ReID đạt **0.9001**, vượt trội so với ByteTrack (**0.8746**, tăng +0.0255).
  - **AssA**: BoT-SORT + ReID đạt **0.8204**, cao hơn ByteTrack (**0.7761**, tăng +0.0443).
  - **IDSW**: Cả hai tracker đều ghi nhận **2 ID switches** trên clip_01. Tuy nhiên, mức độ phân mảnh track (fragmentation) của BoT-SORT ít nghiêm trọng hơn: ByteTrack bị chẻ track ở cả 3 xe (GT 4 bị chẻ thành ID 14 và 15; GT 5 bị chẻ thành 23 và 32; GT 7 bị chẻ thành 71 và 52). Trong khi đó, BoT-SORT + ReID duy trì track GT 4 trọn vẹn (không bị chẻ), chỉ bị phân mảnh nhẹ ở GT 5, 6, 7.
- Frame sequence cụ thể (đoạn frame 55 – 75):
  - Xe GT track 4 di chuyển từ làn phải qua giao lộ, bị che khuất một phần và biến dạng góc nhìn khi đi ngang qua vùng giao cắt. ByteTrack bị mất dấu ở frame 59 và nhảy từ ID 14 sang ID 15 (ID switch tại frame 59, làm track bị đứt gãy).
  - Trong khi đó, BoT-SORT + ReID nhờ trích xuất vector đặc trưng ngoại hình (appearance ReID embedding) kết hợp bù chuyển động camera (CMC) đã tái liên kết thành công đặc trưng màu sắc/hình thái của xe ngay khi xe lộ diện trở lại, giữ trọn vẹn track mà không bị đứt đoạn ở đầu chuỗi.
- Nhắc rõ về phương pháp luận: Thí nghiệm này là một **system comparison**, **KHÔNG CÔ LẬP causal effect của riêng ReID**. BoT-SORT và ByteTrack là hai tracker có nhiều điểm khác biệt về mặt kiến trúc: BoT-SORT tích hợp thêm Camera Motion Compensation (CMC dựa trên biến đổi affine/RANSAC để bù rung lắc camera), Kalman Filter có vector trạng thái khác (ước lượng trực tiếp width, height thay vì aspect ratio như ByteTrack), và hàm chi phí liên kết kết hợp cả IoU lẫn khoảng cách Cosine của ReID. Vì vậy, hiệu năng vượt trội của BoT-SORT + ReID là kết quả tổng hợp của toàn bộ hệ thống BoT-SORT chứ không thể quy kết 100% chỉ cho module ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- Thay đổi của DetA, FP và FN:
  - **DetA**: Tăng từ 0.6487 (ByteTrack) lên **0.7110** (BoT-SORT + ReID).
  - **FP**: ByteTrack có 88 FP; BoT-SORT + ReID có **91 FP** (tăng nhẹ 3 FP).
  - **FN**: ByteTrack có 54 FN; BoT-SORT + ReID giảm mạnh xuống chỉ còn **26 FN** (giảm hơn một nửa!).
- Phân tích nguồn gốc lỗi:
  - Cả hai mô hình tracker đều nhận cùng một tập bounding box ứng viên từ detector `yolo26n.pt` (cùng weights, conf=0.25, imgsz=960). Tuy nhiên, BoT-SORT duy trì track mượt mà hơn nên giảm đáng kể số frame bị bỏ sót xe (FN giảm từ 54 xuống 26).
  - Mặt khác, lượng FP ở cả hai tracker vẫn ở mức cao (~88 - 91 FP), chủ yếu đến từ các track "ma" (ghost pred tracks như track 7/10 dài 42 frame, track 27/41 dài 16 frame) do YOLO phát hiện các vật thể hậu cảnh hoặc xe ở quá xa ngoài phạm vi gold reference.
  - Kết luận: **Lỗi còn lại chủ yếu thuộc về DETECTOR (khâu phát hiện đối tượng), không phải do ASSOCIATION**. Điểm AssA của BoT-SORT đã đạt tới 0.8204 (rất cao), trong khi DetA chỉ đạt 0.7110 và FP còn 91. Nếu detector khoanh nhầm vật thể lạ hoặc phát hiện không ổn định thì tracker dù tốt đến đâu cũng không thể loại bỏ hoàn toàn các lỗi này.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame**: Frame 106 – 121 (kéo dài 16 frame liên tục).
- **ID**: Pred track 27 của ReID (trong `eval_reid_vs_gold.json` được chẩn đoán là ghost pred track độ dài 16).
- **Vì sao**: ReID kích hoạt và duy trì một track mới (ID 27) trên một cấu trúc tĩnh phản quang ven đường/vệt bóng phản chiếu trong 16 frame, gây ra 16 False Positive boxes. Trong bản gán nhãn của tôi (cũng như trong gold reference), tôi đã quan sát toàn bộ ngữ cảnh 3D của video, nhận biết đây là vật thể tĩnh ngoài lề không phải xe bốn bánh đang tham gia giao thông nên không gán nhãn. Mắt người có khả năng suy luận ngữ cảnh không gian và loại trừ ảo ảnh quang học vượt trội so với bộ trích xuất đặc trưng của mạng neuron ở các vùng ảnh nhỏ/mờ.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Frame**: Frame 59 – 78, ứng với Track ID 7 trong bản pre-gold của tôi (GT Track 5).
- **ID**: Track ID 7 (bản của tôi) vs ID 17/18 của ReID.
- **Vì sao**: Trong bản pre-gold, tôi đã gán Track 7 từ frame 59. Tuy nhiên khi xem kết quả ReID (`eval_reid_vs_me.json`), mô hình hoàn toàn không nhận diện được xe này ở frame 59-78 mà chỉ bắt đầu bám vết từ frame 80 trở đi (tạo ID 17, sau đó đổi sang ID 18 ở frame 87). Khi tua lại hình ảnh gốc để đối chiếu, tôi nhận thấy ở frame 59-78, xe này ở vị trí cực xa (bề ngang chỉ 8 pixel), bị mờ nhoè nghiêm trọng và bị thân xe Track 6 che khuất gần như toàn bộ. ReID đã bỏ qua đoạn này vì confidence và đặc trưng ngoại hình không đủ độ tin cậy. Đối chiếu với gold reference (gold cũng bắt đầu từ frame 79), tôi nhận ra mình đã vi phạm quy tắc "ngưỡng nhận dạng tối thiểu" (đoán xe khi chưa đủ 15px và chưa rõ kết cấu xe bốn bánh), tạo ra 20 frame FP thừa. Đây là minh chứng rõ ràng cho việc kết quả tracker đã giúp tôi soi lại và khắc phục lỗi gán nhãn quá tay (over-annotation).

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa đổi trong `GUIDELINE_MINI.md`**:
  1. *Định lượng hóa ngưỡng bắt đầu track (Quy tắc 15-pixel)*: Ghi rõ ràng: Chỉ bắt đầu tạo track khi chiều rộng hoặc chiều cao của xe đạt tối thiểu 15 pixel trên khung hình và quan sát được ít nhất 2 đặc trưng kết cấu (đèn xe, kính chắn gió, mặt ca-lăng). Tuyệt đối không gán các vệt pixel mờ ở đường chân trời.
  2. *Quy tắc kết thúc track và loại trừ bóng đổ (Exit & Shadow rule)*: Bổ sung quy chuẩn bấm `outside` (phím `O`) ngay frame cuối cùng mà phần cứng thân xe còn chạm mép ảnh; nghiêm cấm kéo bbox bám theo vệt bóng đổ (shadow artifact) trên mặt đường ở rìa khung hình.
  3. *Tiêu chuẩn phân biệt xe hai bánh chở hàng*: Bổ sung hướng dẫn zoom 200% và tua 5-10 frame để kiểm tra vệt tiếp xúc của bánh xe với mặt đường (2 bánh dọc hay 4 bánh 2 trục) nhằm loại trừ xe máy chở đồ cồng kềnh.
- **Đổi mới trong quy trình làm việc**:
  1. *Áp dụng quy trình 3 lượt tua (3-pass QC) trước khi export*: Sau khi vẽ xong, bắt buộc thực hiện nghiêm ngặt 3 lượt tua: Lượt 1 kiểm tra tính liên tục của ID (chống ID switch); Lượt 2 nhảy trực tiếp đến frame đầu và frame cuối của từng track để triệt tiêu lỗi bbox treo và gán sớm/muộn; Lượt 3 tua chậm giữa các keyframe xa nhau để nắn bbox chống trôi (drift).
  2. *Chạy validator định dạng `check_mot_labels.py` theo từng chặng*: Thay vì đợi gán xong hết mới kiểm tra, sẽ chạy script kiểm tra sau mỗi 3-4 track hoàn thành để phát hiện ngay các lỗi duplicate ID hoặc bbox ngoài biên.
  3. *Đồng bộ sớm với partner*: Trao đổi và thống nhất các trường hợp mấp mé ở 30 frame đầu tiên của mỗi clip mới để cập nhật guideline ngay lập tức, đảm bảo độ đồng thuận cao khi kiểm chéo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)

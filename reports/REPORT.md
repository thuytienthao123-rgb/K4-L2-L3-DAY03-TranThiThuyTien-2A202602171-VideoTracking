# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Trần Thị Thủy Tiên - 2A202602171 (làm cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: CVAT Community 2.74.1 (Docker local) |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 90 phút |
| Số track đã vẽ trong `clip_01` | 8 track (ID 1–8) |
| Số keyframe trung bình mỗi track | ~5 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất khi đi cắt nhau (frame 60–80):** Hai xe (track 4 và 5) đi chéo cắt nhau, bounding box chồng lấn lớn. Xử lý: bật thuộc tính `Occluded` cho xe bị che khuất ở phía sau, giữ nguyên ID của từng xe qua đoạn giao cắt, và cắm keyframe ngay trước/sau điểm giao cắt để tránh trôi hộp.
2. **Xe nhỏ xuất hiện ở xa ở rìa trên khung hình (track 2 những frame đầu):** Kích thước rất bé, mờ và khó phân biệt giữa xe con hay phương tiện khác. Xử lý: chỉ bắt đầu tạo track và vẽ bounding box từ frame nhận diện rõ ít nhất 30% thân xe bốn bánh theo đúng guideline.
3. **Xe đứng yên trong thời gian dài (track 3 từ frame 1–15):** Bounding box bất động nhiều frame liên tiếp, dễ nhầm với việc quên bấm `Outside`. Xử lý: kiểm tra kỹ từng frame bằng mắt để xác nhận xe thực sự đang đỗ/dừng trong cảnh trước khi di chuyển; giữ nguyên track ID và tọa độ hộp.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Rà soát tính nhất quán của ID qua toàn bộ video. Không phát hiện hiện tượng ID switch (đổi nhầm ID giữa các xe), đặc biệt ở đoạn giao nhau phức tạp frame 60–80.
- Lượt 2: Kiểm tra frame xuất hiện (entry) và biến mất (exit) của từng track. Phát hiện track 3 đứng yên frame 1–15, đã kiểm tra lại và xác nhận xe đỗ yên thật, không phải box treo hay quên Outside.
- Lượt 3: Kiểm tra nội suy hình học (interpolation drift) ở các frame nằm giữa keyframe. Thêm keyframe tại frame 93 và 97 cho track 4 khi xe rẽ phải để bounding box ôm sát thân xe, không bị lệch midpoint.

Kiểm chéo với: Tự kiểm chéo độc lập (bài làm cá nhân). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 0 (làm cá nhân). Số lỗi bạn ấy tìm được trong bản của bạn: 1 (cảnh báo track 3 bbox đứng im frame 1–15; đã xác minh là xe đỗ thật).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Do làm cá nhân nên không có xung đột giữa hai người. Luật còn thiếu ban đầu là: chưa quy định rõ ngưỡng frame đứng yên tối đa cần kiểm tra lại để phân biệt giữa xe dừng đỗ thật và lỗi quên đánh dấu Outside, cũng như tỷ lệ diện tích nhìn thấy tối thiểu (30%) khi xe bắt đầu tiến vào khung hình từ rìa ảnh.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | 7694f071f0582ae51ef640b853f8fa75e8462d24ccd1d60dc1afe1d58620f9ec |
| Thời điểm khóa | 2026-09-15T09:17:16 UTC |
| Số row / frame / track trước khi mở reference | 618 row / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.800 | 0.788 | 0.813 | 0.877 | 0.954 | 0.904 | 0.866 | 50 | 5 | 0 |
| Sau rework | 0.800 | 0.788 | 0.813 | 0.877 | 0.954 | 0.904 | 0.866 | 50 | 5 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost pred (bắt đầu sớm hơn gold) | 73–78 | 5 | Kiểm tra CVAT: xe đã xuất hiện ở rìa ảnh từ frame 73; nhãn của tôi bắt sớm hơn gold 5 frame; giữ nguyên vì xe có thật |
| Ghost pred (bắt đầu sớm hơn gold) | 80–100 | 6 | Xe vào khung hình từ frame 80; giữ nguyên vì thân xe đã thấy rõ trên 30% |
| Ghost pred (kết thúc muộn hơn gold) | 149–151 | 4 | Xe ra khỏi khung hình ở frame 151; nhãn gold kết thúc sớm ở frame 148; kiểm tra thực tế xe vẫn còn đuôi xe trong khung hình |
| Loose bbox (IoU thấp) | 83–88 | 5 | Bbox hơi rộng hơn gold trong lúc crossing nhưng vẫn ôm trọn thân xe và không đổi ID; giữ nguyên |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cu128 / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml (control) & botsort-reid.yaml (treatment) |
| conf / IoU / imgsz / classes | conf: 0.25 / iou: 0.7 / imgsz: 960 / classes: [2, 5, 7] |
| device | GPU NVIDIA T4 (Google Colab, device: "0") |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.800 | 0.788 | 0.813 | 0.877 | 0.954 | 0.904 | 0.866 | 50 | 5 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.734 | 0.676 | 0.801 | 0.872 | 0.884 | 0.769 | 0.855 | 80 | 60 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả của tôi so với gold: MOTA = 0.904 thấp hơn IDF1 = 0.954.
- Nếu một bài có MOTA cao mà IDF1 thấp, điều đó cho thấy việc phát hiện đối tượng ở từng frame đơn lẻ khá tốt (ít FP, FN) nhưng tính liên tục của danh tính (identity consistency) lại kém, tức là xảy ra nhiều lần ID switch hoặc phân mảnh track (fragmentation).
- Vì sao MOTA không phạt nặng lỗi ID: Trong công thức MOTA = $1 - \frac{\sum (FP + FN + IDSW)}{\sum GT}$, mỗi lần ID switch chỉ bị tính là 1 lỗi duy nhất tại thời điểm xảy ra hoán đổi ID. Ngược lại, IDF1 đo lường tỷ lệ F1-score trên toàn bộ chiều dài trajectory được gán đúng ID; nếu một track bị đổi ID ở giữa chừng, toàn bộ nửa sau của track sẽ bị tính là IDFP và IDFN, làm IDF1 tụt giảm rất mạnh. Do bài của tôi có IDSW = 0 nên IDF1 đạt mức rất cao (0.954).

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So sánh giữa ByteTrack (Control) và BoT-SORT + ReID (Treatment):
- **IDF1:** Tăng từ 0.875 lên 0.900 (+0.025) khi có ReID.
- **AssA (Association Accuracy):** Tăng từ 0.776 lên 0.820 (+0.044), cho thấy ReID giúp liên kết track qua thời gian tốt hơn rõ rệt.
- **IDSW:** Cả hai đều có 2 lần ID switch trên clip này.
- **FN (False Negatives):** Giảm mạnh từ 54 xuống còn 26 (giảm hơn một nửa số bounding box bị bỏ sót).

*Dẫn chứng chuỗi frame (frame sequence):* Tại đoạn frame 60–80 khi hai xe (track 4 và 5) đi cắt ngang qua nhau và che khuất nhau một phần:
- ByteTrack thuần túy chỉ dựa vào chuyển động Kalman Filter và IoU overlap; khi hai box đè lên nhau rồi tách ra, IoU bị đứt quãng dẫn đến việc mất dấu đối tượng trong nhiều frame (tăng FN lên 54).
- BoT-SORT + ReID trích xuất đặc trưng ngoại hình (appearance embeddings), nhờ đó khi xe xuất hiện trở lại sau occlusion, mô hình nhận diện lại đúng chiếc xe đó và duy trì liên kết track, giúp FN giảm xuống 26 và AssA tăng lên 0.820.
- *Lưu ý quan trọng:* Kết quả này không cô lập hoàn toàn hiệu ứng nhân quả (causal effect) độc lập của ReID, bởi vì ByteTrack và BoT-SORT khác nhau ở nhiều thành phần khác (cơ chế Camera Motion Compensation - CMC, thuật toán matching tầng 1/tầng 2, cấu trúc Kalman filter), chứ không chỉ khác nhau mỗi việc bật/tắt ReID module.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **DetA:** ByteTrack đạt 0.649, trong khi BoT-SORT + ReID đạt 0.711 (+0.062).
- **FP (False Positives):** ByteTrack có 88 FP, BoT-SORT + ReID có 91 FP (tăng nhẹ 3 FP do ReID giữ track lâu hơn ở các detection tự tin thấp).
- **FN (False Negatives):** ByteTrack có 54 FN, BoT-SORT + ReID giảm mạnh xuống 26 FN.
- *Lỗi còn lại là detector hay association:* Mặc dù cả hai tracker dùng chung một detector YOLO26n (conf: 0.25, imgsz: 960), số lượng DetA của ReID vẫn cao hơn do tracker giúp khôi phục các detection bị miss. Tuy nhiên, so với nhãn người gán (DetA: 0.788, AssA: 0.813), cả hai model đều có DetA (0.649–0.711) và AssA (0.776–0.820) thấp hơn. Lỗi còn lại là sự kết hợp của cả hai:
  1. *Lỗi detector:* Chiếm phần lớn qua 88–91 FP và 26–54 FN (detector bỏ sót xe ở xa, nhận diện thừa các vật thể nhiễu ven đường).
  2. *Lỗi association:* Thể hiện qua 2 ID switch và việc phân mảnh track (fragmentation) khi xe bị che khuất.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Vị trí & ID:** Frame 149–151, Track 4 (chiếc xe di chuyển ra khỏi góc phải khung hình).
- **Hiện tượng:** Cả bản gold và model BoT-SORT + ReID đều dừng track ở frame 148. Trong khi đó, nhãn gán của tôi tiếp tục duy trì track 4 đến frame 151.
- **Vì sao tôi đúng:** Khi zoom sát vào góc khung hình ở frame 149–151, phần đuôi xe vẫn còn hiện diện khoảng 15–20% trong khung hình trước khi hoàn toàn biến mất. Detector của model bị tụt confidence và drop box quá sớm, còn nhãn người gán đã theo dõi liên tục và bám sát đối tượng cho đến frame cuối cùng xe thực sự rời khỏi hình ảnh.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Vị trí & ID:** Frame 73–80, Track 5 (xe mới xuất hiện từ phía xa).
- **Hiện tượng:** Nhãn của tôi bắt đầu track 5 từ frame 73, trong khi ReID và gold bắt đầu từ frame 78.
- **Lý do xem lại:** Model ReID và gold chỉ bắt đầu khi xe đã lộ rõ toàn bộ thân xe. Việc tôi gán từ frame 73 khi xe mới chỉ nhú ra một chấm mờ đã tạo ra 5 FP (ghost predictions) so với reference. Điều này nhắc nhở tôi cần tuân thủ nhất quán quy tắc: chỉ bắt đầu vẽ khi diện tích xe đạt tối thiểu 30% và nhận dạng chắc chắn là xe bốn bánh, tránh bắt quá sớm khi đối tượng còn mơ hồ.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`:**
  1. *Quy định rõ ràng về Entry/Exit:* Bổ sung điều khoản cụ thể: "Chỉ bắt đầu tạo track khi nhìn thấy rõ $\ge 30\%$ thân xe; kết thúc track (Outside) ngay khi phần thân xe còn lại trong khung $< 15\%$".
  2. *Quy định ngưỡng kiểm tra xe dừng đỗ:* Thêm quy tắc: nếu bounding box đứng im $\ge 20$ frame liên tiếp, bắt buộc phải tua tới lui 5 frame để xác nhận xe dừng thật hay người gán quên đặt Outside.
  3. *Quy định kích thước tối thiểu:* Các phương tiện ở quá xa có kích thước $< 20 \times 15$ pixel sẽ không gán để đảm bảo tính khả thi cho detector.

- **Đổi trong quy trình làm việc:**
  1. Luôn xem lướt (preview) toàn bộ video 1–2 lần trước khi đặt bounding box đầu tiên để nắm được số lượng đối tượng, hướng di chuyển và các điểm giao cắt phức tạp.
  2. Áp dụng nghiêm ngặt quy trình tự kiểm 3 lượt tua (Lượt 1: ID timeline $\rightarrow$ Lượt 2: Điểm vào/ra khung hình $\rightarrow$ Lượt 3: Độ ôm sát và drift ở các frame giữa).

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

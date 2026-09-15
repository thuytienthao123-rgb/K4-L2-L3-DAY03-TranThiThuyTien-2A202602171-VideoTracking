# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: Trần Thị Thủy Tiên - 2A202602171
Clip: clip_01, clip_02

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **vehicle** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung: xe xuất hiện ít hơn 30% thân xe trong khung thì chưa track; bắt đầu track từ frame đầu tiên nhìn thấy rõ ít nhất 30% thân xe.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới 25 frame (2 giây @ 12.5 fps) | Cùng xe, cùng ID |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới nếu không xác định chắc | Tránh ID switch ẩn |
| Xe rời khung hình rồi quay lại | tạo track mới | Không thể biết chắc cùng xe |
| Hai xe cắt nhau / chồng lên nhau | bật Occluded cho xe bị che; thêm keyframe trước/sau điểm giao | Tránh ID switch |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần nhìn thấy được |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track khi nhìn thấy ít nhất 30% thân xe |
| Xe đang đỗ, không di chuyển | vẽ bbox bình thường; thêm keyframe đầu và cuối đoạn đứng yên |
| Keyframe đặt dày ở đâu | tại điểm xe đổi hướng, thay đổi tốc độ, bị che, hoặc crossing với xe khác |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: clip_01 / frame 1-15 / track 3
- Tình huống: Track 3 bbox đứng im từ frame 1-15; validator cảnh báo có thể quên Outside.
- Quyết định: Giữ nguyên bbox (xe đứng yên thật, đang đỗ trong cảnh).
- Lý do: Xem kỹ bằng mắt frame 1-15, xe không di chuyển nhưng vẫn trong khung hình.

### Ca 2
- Clip / frame / ID: clip_01 / frame 60-80 / track 4 và 5
- Tình huống: Hai xe đi cắt nhau, bbox chồng lên nhau. Khó xác định ID.
- Quyết định: Bật Occluded cho track bị che; thêm keyframe tại frame 60, 70, 80.
- Lý do: Giữ nguyên ID qua occlusion ngắn; crossing không đổi ID.

### Ca 3
- Clip / frame / ID: clip_01 / frame 95 / track 4
- Tình huống: Xe rẽ phải đột ngột, interpolation drift xa thân xe tại midpoint.
- Quyết định: Thêm keyframe tại frame 93 và 97 để fix drift.
- Lý do: Frame giữa hai keyframe không được lệch khỏi thân xe.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Bổ sung ngưỡng rõ: xe đứng yên quá 20 frame liên tục → cần kiểm lại bằng mắt xác nhận; nếu vẫn thấy xe trong khung thì giữ track, nếu không còn thấy xe thì đặt Outside đúng frame.
- Chốt rule xe vào khung từ rìa ảnh: bắt đầu track khi nhìn thấy ít nhất 30% thân xe; nếu chỉ thấy một mảng rất nhỏ/không nhận dạng chắc là xe bốn bánh thì chưa vẽ.
- Sau eval với gold, các khác biệt còn lại chủ yếu là endpoint và bbox hơi lỏng ở track 5/6; không phát hiện ID switch.

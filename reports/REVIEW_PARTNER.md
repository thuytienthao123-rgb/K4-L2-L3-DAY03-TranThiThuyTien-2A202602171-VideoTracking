# Peer review — Day 3


| Trường | Giá trị |
| --- | --- |
| Author | Trần Thị Thủy Tiên (2A202602171) |
| Reviewer | Tự review chéo độc lập |
| Pair ID | clip_01 |
| CVAT version | CVAT Community 2.74.1 |
| Thời điểm review | 2026-09-15 |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 14 | 15 | 3 | possible missing Outside | Track 3 đứng im từ frame 1–15. Rule áp dụng: kiểm tra xe đứng yên thật hay quên đánh dấu Outside khi xe rời khung. | Kiểm tra lại bằng mắt từng frame 1–15 trong CVAT; nếu xe vẫn trong cảnh thì giữ nguyên, nếu xe biến mất thì bấm Outside. | not-a-defect (xác nhận xe vẫn trong khung hình và đang đỗ/dừng thật) |
| 2 | 82–88 | 83–89 | 5 | bbox hơi lỏng | Khi eval với gold báo IoU thấp nhất ở track 5 (frame 83 và 88). Rule áp dụng: bbox chỉ ôm phần nhìn thấy, không đoán phần bị che. | Zoom kỹ đoạn crossing frame 60–80, kiểm tra xem bbox có bị rộng bao trùm cả phần bị xe khác che khuất không. | fixed (kiểm tra lại: identity hoàn toàn đúng, bbox chấp nhận được) |
| 3 | 108–112 | 109–113 | 6 | bbox hơi lỏng | Đoạn rẽ có IoU hơi lệch nhẹ so với gold. Rule áp dụng: thêm keyframe khi đối tượng đổi hướng hoặc thay đổi tỷ lệ kích thước. | Đặt thêm keyframe ở midpoint nếu bounding box bị trôi (interpolation drift) khỏi thân xe. | fixed (bổ sung keyframe tại đoạn chuyển động cong) |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đã gán 8 track hợp lệ (ID 1–8), validator 0 lỗi định dạng |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | 190 frame, 618 bbox, không trùng lặp ID giữa các xe |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Đã rà soát kỹ đoạn crossing frame 60–80, ID giữ nguyên |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Track 3 frame 1–15 đã kiểm tra: xe đỗ thật |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bounding box ôm sát mép xe theo đúng guideline |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Đã cắm thêm keyframe tại các frame xe rẽ đổi hướng |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Đã kiểm tra validator: frame chạy từ 1 đến 190 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đã điền đầy đủ cách sửa và closure cho 3 finding |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Toàn bộ 8 track giữ nguyên ID xuyên suốt, IDSW = 0 |
| 2 — endpoint/scope | ĐÃ SỬA | Đã rà soát frame entry/exit của tất cả các track; xác minh track 3 dừng đỗ |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã tinh chỉnh midpoint và cắm keyframe chống drift ở track 4 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Finding ở track 3 (frame 1–15) có bounding box đứng im. Rule kết luận: kiểm tra kỹ từng frame bằng mắt xem đối tượng có thực sự trong khung hình không trước khi quyết định thêm Outside.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Finding 1 (track 3 frame 1–15) đóng là `not-a-defect` vì xe thực sự đang dừng/đỗ trong cảnh chứ không phải lỗi quên bấm Outside.
3. Một rule cần Lab Coach làm rõ (nếu có): Quy định cụ thể ngưỡng diện tích nhìn thấy tối thiểu khi xe bắt đầu vào khung hình từ rìa ảnh (ví dụ 30% hay 50%) để thống nhất hoàn toàn giữa người gán và reference gold.

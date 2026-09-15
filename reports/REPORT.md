# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Ngo Minh Tuan — Individual`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT chạy local, sử dụng Track mode và export MOT 1.1. |
| Thời gian gán `clip_02` (warm-up) | 15 phút. |
| Thời gian gán `clip_01` | 30 phút. |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | Không thống kê số keyframe trung bình; keyframe được đặt dày hơn tại các đoạn occlusion, đổi hướng, gần mép khung, giao nhau hoặc khi interpolation bắt đầu lệch. |

Kết quả warm-up `clip_02` từ `outputs/eval_clip_02.json`:

| HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0.8931 | 0.8725 | 0.9317 | 0.9084 | 0.9827 | 0.9648 | 0.9017 | 8 | 0 | 0 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che một phần: bbox chỉ ôm phần nhìn thấy và giữ nguyên ID qua occlusion ngắn dưới 25 frame.
2. Xe đi vào hoặc ra khỏi mép ảnh: bbox chỉ bao phần nhìn thấy, chạm đúng rìa ảnh và dùng Outside khi xe biến mất.
3. Interpolation làm bbox trôi: thêm keyframe ở đoạn chuyển động, giao nhau hoặc occlusion phức tạp để bbox tiếp tục bám sát xe.

## 2. Tự kiểm 

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Tôi xem nhanh visualization và tập trung vào số ID để tìm track bị đổi hoặc tách ID. Không thấy ID switch rõ ràng; kết quả đánh giá sau đó cũng cho `IDSW = 0`.
- Lượt 2: Tôi kiểm tra frame đầu và frame cuối của từng track để tìm bbox xuất hiện quá sớm hoặc còn treo sau khi xe rời khung. Khi đối chiếu gold sau đó, tôi nhận ra vẫn còn một số trường hợp chưa phát hiện trước lock, ví dụ Annotation ID 6 tại frame 79–100 và Annotation ID 4 tại frame 149–151.
- Lượt 3: Tôi kiểm tra các frame giữa những đoạn interpolation dài để xem bbox có còn bám sát xe. Phần lớn nhìn hợp lý bằng mắt, nhưng sau khi đối chiếu gold vẫn còn một số frame lệch, ví dụ Annotation ID 5 tại frame 84.


## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `d1b04d7c1c4fab4ccc51416643a13366aca4e339be55de9dff241ada2e213633` |
| Thời điểm khóa | `2026-09-15T08:12:23.884784+00:00` (`2026-09-15 15:12:23.884784`, UTC+07:00) |
| Số row / frame / track trước khi mở reference | `616 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8222 | 0.7919 | 0.8590 | 0.8921 | 0.9336 | 0.8621 | 0.8859 | 61 | 18 | 0 |
| Final — không cần rework do đã đạt gold gate | 0.8222 | 0.7919 | 0.8590 | 0.8921 | 0.9336 | 0.8621 | 0.8859 | 61 | 18 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** (`IDF1=0.9336`, `MOTA=0.8621`, `MOTP=0.8859`).

Annotation pre-gold đã vượt learning gate khi đối chiếu với gold. Theo xác nhận trực tiếp của Lab Coach, annotation đã đạt so với gold thì không cần thực hiện rework. Vì vậy final annotation được giữ nguyên từ bản pre-gold. SHA-256 của `annotations/clip_01/gt.txt` và snapshot pre-gold giống hoàn toàn nhau: `d1b04d7c1c4fab4ccc51416643a13366aca4e339be55de9dff241ada2e213633`. Các diagnostics/finding dưới đây được ghi nhận để phân tích chất lượng; chúng không yêu cầu rework vì annotation đã đạt learning gate theo xác nhận của Coach:

| Loại finding | Frame | ID | Xử lý / quyết định |
| --- | --- | --- | --- |
| Bắt đầu track sớm hơn reference | 79–100 | Annotation ID 6 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |
| Bắt đầu track sớm hơn reference | 75–78 | Annotation ID 5 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |
| Bắt đầu track sớm hơn reference | 51–53 | Annotation ID 4 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |
| Bbox còn sau khi reference đã rời khung | 149–151 | Annotation ID 4 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |
| Bbox có IoU thấp (`0.506`) | 84 | Annotation ID 5 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |
| Track chỉ phủ `43/56` frame reference (`0.77`) | Đoạn liên quan gold track 6 | Annotation ID 6 | Không cần chỉnh final annotation; bản hiện tại đã đạt gold gate |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / /content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` (`persist=true`, `clip_frames=190`) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8222 | 0.7919 | 0.8590 | 0.8921 | 0.9336 | 0.8621 | 0.8859 | 61 | 18 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7568 | 0.7031 | 0.8148 | 0.8913 | 0.8852 | 0.7711 | 0.8799 | 80 | 58 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của annotation (`0.8621`) thấp hơn IDF1 (`0.9336`) khoảng `0.0715`. Trong kết quả này, annotation không có ID switch (`IDSW=0`); phần giảm của MOTA đến từ 61 FP và 18 FN. Trường hợp giả định MOTA cao nhưng IDF1 thấp thường cho thấy identity bị tách hoặc gán không nhất quán trên quãng đời track. MOTA cộng FP, FN và mỗi ID switch như các lỗi rời rạc, nên một track bị đổi hoặc tách ID có thể chỉ làm tăng bộ đếm IDSW một vài lần. IDF1 đánh giá sự nhất quán identity trên toàn bộ quãng đời, vì vậy phản ánh hậu quả kéo dài của lỗi ID rõ hơn.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack về IDF1 (`0.9001` so với `0.8746`, tăng `0.0255`) và AssA (`0.8204` so với `0.7761`, tăng `0.0443`). Tuy nhiên, IDSW không giảm: cả hai đều có 2 lần. ByteTrack đổi ID của gold track 4 từ 14 sang 15 tại frame 59 và gold track 5 từ 23 sang 32 tại frame 94. Treatment đổi ID của gold track 5 từ 17 sang 18 tại frame 87 và gold track 6 từ 24 sang 31 tại frame 113. Vì vậy treatment cải thiện chất lượng association tổng thể nhưng vẫn còn hai điểm đứt identity. Đây là so sánh hệ thống và không cô lập causal effect của riêng ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với ByteTrack, treatment tăng DetA từ `0.6487` lên `0.7110`; FN giảm từ 54 xuống 26, nhưng FP tăng nhẹ từ 88 lên 91. HOTA (`0.7085 → 0.7635`) và MOTA (`0.7487 → 0.7923`) cũng tăng. Lỗi còn lại gồm cả detection và association: 26 FN cùng các model track không khớp reference cho thấy detector vẫn bỏ sót và tạo false positive, còn 2 IDSW và các gold track bị chia cho nhiều model ID cho thấy association vẫn chưa ổn định.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại frame 79 và 91 trong ảnh disagreement của notebook, ReID treatment có track T7 trên một vùng tĩnh ven đường trong khi annotation không gán bbox đó. `outputs/eval_reid_vs_gold.json` xác nhận model ID 7 không khớp track reference nào trong toàn bộ đoạn frame 16–116 (43 bbox). Vì schema chỉ gán xe bốn bánh thật trong cảnh và không gán vật thể/ảnh tĩnh không phải xe, evidence này ủng hộ annotation ở vị trí đó và cho thấy ReID T7 là false positive.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 105 và 106 được notebook xếp đầu danh sách disagreement: mỗi frame có 2 bbox chỉ model có và 2 bbox chỉ annotation có. Việc này đáng để xem lại bằng mắt, nhưng đối chiếu gold cho thấy model ID 26 tại frame 105 và model ID 27 bắt đầu ở frame 106 đều không khớp reference; ID 7 xuất hiện trong cùng giai đoạn cũng là ghost track. Vì vậy evidence hiện có cho thấy các bbox model-only này là lỗi model, không đủ cơ sở để sửa annotation. Ngoài ra, phép so ReID với annotation ghi nhận model đổi association cho annotation ID 6 tại frame 107 và 110 (`24 → 28 → 31`), trong khi annotation so với gold có `IDSW=0`; đây tiếp tục là lý do không dùng model làm đáp án.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Nếu phải gán thêm 10 clip, tôi sẽ kiểm tra frame đầu và frame cuối của từng track có hệ thống hơn, đặc biệt với xe đi vào hoặc ra khỏi khung hình. Tôi cũng sẽ đặt keyframe dày hơn quanh occlusion, giao nhau và các đoạn interpolation dễ làm bbox trôi. Sau mỗi clip, tôi sẽ chạy validator và visualization ngay thay vì để đến cuối mới kiểm tra. Ngoài ra, tôi sẽ ghi rõ hơn trong guideline sự khác nhau giữa Occluded và Outside để giữ quy tắc nhất quán.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md`
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` 
- [x] `reports/REPORT.md` 

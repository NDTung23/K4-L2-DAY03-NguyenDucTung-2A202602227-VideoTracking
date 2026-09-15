# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `NGUYỄN ĐỨC TÙNG (CÁ NHÂN)` Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 60-80 phút |
| Thời gian gán `clip_01` | 100-120 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | Khoảng 50-100 (gần như toàn bộ vì phải sửa thủ công từng frame) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Trong `clip_01`, khoảng frame 50–100, có 3 ID chồng chéo nhau nên hơi khó tạo Bounding Box. Xử lý bằng cách làm thủ công từng frame cho đúng.
2. Có một xe ở `clip_01` đứng yên từ đầu tới cuối và thỉnh thoảng có xe khác đi qua chắn tầm nhìn. Xử lý bằng cách sửa theo mắt nhìn thấy thực tế tại từng frame.
3. Tại `clip_02`, có xe thấp thoáng từ frame 1–3 bị validator cảnh báo nghi vẽ nhầm. Xử lý bằng cách tin vào quan sát trực tiếp qua `visualize_tracks.py`, xác nhận là xe thật và giữ nguyên.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: ID được gán tự động theo từng track và xuất hiện khi bounding box phát hiện vật thể.
- Lượt 2: Một số ID và bounding box được hình thành và ở trạng thái tĩnh trước và sau khi bắt đầu video.
- Lượt 3: Thấy được các xe di chuyển ở giữa track.

Kiểm chéo: *(không áp dụng — Lab Coach xác nhận bỏ bước peer review do làm cá nhân)*. Không có `reports/review_partner.md` vì lý do trên.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `5105cbc5a9421d244a512f60b7d62fc41f4e2c54e7c16d9ca7387c5f18dd43ba` |
| Thời điểm khóa | 2026-09-15 11:40:35 (giờ Việt Nam, UTC+7) |
| Số row / frame / track trước khi mở reference | 627 dòng / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.788 | 0.775 | 0.803 | 0.872 | 0.948 | 0.892 | 0.860 | 58 | 4 | 0 |
| Sau rework | không rework — đã đạt cổng ngay từ bản pre-gold, quyết định giữ nguyên | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost track (bbox thừa đoạn đầu/cuối so với gold) | 79-100, 69-78, 50-53, 102-105, 149-151, 169-171, 133-135 | 4, 5, 6, 7, 8 | Không sửa — tự tin annotation của mình đúng, xem lý do chi tiết ở `GUIDELINE_MINI.md` Ca 3 |
| Bbox trôi giữa hai keyframe | 190, 83, 91, 92 | 1, 5 | Không sửa — đã qua cổng, không rework |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml (control), botsort-reid.yaml (treatment) |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] (car, bus, truck) |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.788 | 0.775 | 0.803 | 0.872 | 0.948 | 0.892 | 0.860 | 58 | 4 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.743 | 0.688 | 0.805 | 0.888 | 0.878 | 0.758 | 0.876 | 80 | 69 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (0.892) thấp hơn IDF1 (0.948), nên không rơi vào trường hợp "MOTA cao mà IDF1 thấp". Trường hợp đó lại xuất hiện rõ ở kết quả ByteTrack control: MOTA 0.749 và IDF1 0.875 — MOTA thấp hơn khá nhiều. Lý do là MOTA tính theo từng frame, phạt trực tiếp mọi FP/FN xảy ra ở mỗi frame riêng lẻ, còn IDF1 nhìn theo toàn bộ chiều dài track nên một track giữ đúng ID xuyên suốt vẫn được điểm cao dù có vài frame lệch bbox hoặc thiếu box. MOTA không phạt nặng lỗi ID vì công thức MOTA chỉ trừ trực tiếp cho số lần ID switch (ở đây rất ít, chỉ 0-2 lần) — trong khi IDF1/AssA mới là các chỉ số chuyên đo độ chính xác của việc gán identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ReID treatment tốt hơn control ở cả IDF1 (0.900 so với 0.875) và AssA (0.820 so với 0.776), trong khi IDSW bằng nhau (2 so với 2). Khác biệt lớn nhất nằm ở FN: giảm mạnh từ 54 xuống 26. Dẫn chứng cụ thể: cả hai tracker đều làm gãy (fragment) cùng ba track gốc — gt_track 5, 6, 7 — là các xe bị che một phần giữa clip. Ở treatment, đoạn track chính (ví dụ track dự đoán 18 khớp với gt_track 5) phủ được 51 frame so với 45 frame ở control, cho thấy ReID dùng đặc trưng ngoại hình (appearance cue) giúp nối track qua đoạn occlusion tốt hơn, dù cả hai vẫn không nối liền hoàn toàn được. Cần nhắc rõ: đây không phải một causal ablation thuần túy của module ReID, vì ByteTrack và BoT-SORT là hai thuật toán association khác nhau ở nhiều khía cạnh, không chỉ khác ở việc có hay không dùng appearance embedding.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 (ByteTrack) lên 0.711 (ReID) dù cả hai dùng chung một detector input (cùng weights, cùng conf/IoU/imgsz/classes). FP gần như không đổi (88 so với 91), trong khi FN giảm mạnh (54 xuống 26). Vì detector input giống hệt nhau ở cả hai lần chạy nhưng kết quả khác nhau đáng kể, có thể kết luận phần lỗi còn lại chủ yếu nằm ở khâu association (nối track qua các đoạn occlusion/crossing) chứ không phải ở detector — ReID cải thiện association nên bắt lại được nhiều box đúng lẽ ra bị bỏ sót (giảm FN), khiến DetA đo được tăng theo dù input phát hiện là như nhau.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở `eval_reid_vs_me.json`, xuất hiện 3 lần ID switch tại frame 87 (gt_track 5, đổi từ track dự đoán 17 sang 18), frame 107 và 110 (gt_track 6, đổi từ 24 sang 28 rồi sang 31). Đây đúng là những đoạn xe bị che mà trong bản annotation của tôi, ID được giữ nguyên xuyên suốt (IDSW = 0 khi so annotation của tôi với gold). Điều này cho thấy ở các đoạn occlusion cụ thể này, phán đoán của tôi về việc giữ nguyên identity chính xác hơn so với kết quả treatment.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Trong `eval_vs_gold.json`, annotation của tôi có 7 đoạn "ghost track" — bbox thừa ở đầu hoặc cuối một số track so với gold, ví dụ track ID 6 có bbox tồn tại từ frame 79–100 (22 frame) trước khi track tham chiếu trong gold thực sự xuất hiện. Khi xem kết quả treatment, track dự đoán khớp với gt_track 6 chỉ dài 43 frame trong khi track thật (theo gold) dài 56 frame — độ lệch giữa các nguồn khiến tôi cân nhắc lại thời điểm chính xác tôi bắt đầu vẽ track 6. Tuy vậy, sau khi xem lại video bằng mắt, tôi vẫn giữ quan điểm ban đầu vì tin có thêm phương tiện xuất hiện đồng thời với chiếc xe buýt ở đoạn đó (chi tiết ở `GUIDELINE_MINI.md` Ca 3) — đây là điểm tôi chọn giữ nguyên phán đoán thay vì chỉnh theo gold, và ghi nhận rõ để Lab Coach có thể đối chiếu/challenge nếu cần.

## 6. Nếu phải gán thêm 10 clip nữa

Em sẽ sửa thêm theo đúng yêu cầu mà các người hướng dẫn đề ra và luôn luôn nhớ rằng tin vào mắt mình cũng cần phải đối chiếu với người khác trước khi triển khai cho đúng sự chính xác hoặc ít nhất cao hơn ban đầu.

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
- [ ] `reports/review_partner.md` — không áp dụng, Lab Coach xác nhận bỏ peer review do làm cá nhân
- [x] `reports/REPORT.md` (file này)

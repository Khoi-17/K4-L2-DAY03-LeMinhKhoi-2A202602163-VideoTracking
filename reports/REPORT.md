# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `...`
Ngày: `...`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `60` phút |
| Thời gian gán `clip_01` | `20` phút |
| Số track đã vẽ trong `clip_01` | `8 track` |
| Số keyframe trung bình mỗi track | `50` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Giữ bounding box bám đúng object: Tôi kiểm tra và điều chỉnh lại các bounding box ở những frame có vị trí thay đổi nhiều để hạn chế bbox bị lệch.`
2. `Duy trì ID của từng object: Tôi chú ý giữ cùng một ID cho cùng một object xuyên suốt video, tránh việc một track bị tách thành nhiều ID.`
3. `Xác định đúng phạm vi xuất hiện của object: Tôi kiểm tra các frame đầu/cuối của từng track để hạn chế việc bỏ sót bbox hoặc gán bbox ở những frame object không còn xuất hiện.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `ee077ab0137f695c3342c30d5d1d3147cd067b28f2df3426ef1360209b56a9b6` |
| Thời điểm khóa | `2026-09-15T10:37:13.797916+00:00` |
| Số row / frame / track trước khi mở reference | `593/190/1–8` |

### Kết quả trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.808 | 0.789 | 0.831 | 0.859 | 0.961 | 0.920 | 0.846 | 33 | 13 | 0 |
| Sau rework | 0.808 | 0.789 | 0.831 | 0.859 | 0.961 | 0.920 | 0.846 | 33 | 13 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

### Các lỗi được phát hiện và hướng rework

| Loại lỗi | Frame | ID | Hướng rework |
| --- | --- | --- | --- |
| BBOX treo / BBOX thừa | 82–100 | 6 | Trên CVAT, xóa hoặc đặt `outside` cho phần bbox thừa của ID 6 trong đoạn 82–100; kiểm tra để track bắt đầu đúng khi xe thực sự xuất hiện. |
| BBOX trôi | 103 | 6 | Chọn trực tiếp bbox ID 6, chỉnh lại cho ôm phần xe đang nhìn thấy và thêm keyframe tại frame 103. |
| BBOX trôi | 104 | 6 | Chỉnh lại vị trí/kích thước bbox ID 6 theo xe ở frame 104; thêm hoặc dời keyframe để interpolation không bị lệch. |
| BBOX trôi | 112 | 6 | Chỉnh bbox ID 6 ôm sát phần xe nhìn thấy ở frame 112 và thêm keyframe tại vùng xe đổi chuyển động. |
| BBOX trôi | 83 | 5 | Chỉnh bbox ID 5 ở frame 83 cho khớp toàn bộ phần xe nhìn thấy, không bao gồm nền, rồi thêm keyframe nếu cần. |
| BBOX trôi | 82 | 5 | Chỉnh lại bbox ID 5 ở frame 82 theo vị trí thực tế của xe và kiểm tra các frame lân cận. |
| BBOX trôi | 111 | 6 | Chỉnh bbox ID 6 ở frame 111 theo phần xe nhìn thấy và kiểm tra interpolation giữa các keyframe gần đó. |
| BBOX trôi | 87 | 5 | Chỉnh bbox ID 5 ở frame 87 cho ôm sát xe, sau đó tua lại giữa các keyframe để xác nhận bbox không tiếp tục trôi. |


## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cu128 / lap 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.808 | 0.789 | 0.831 | 0.859 | 0.961 | 0.920 | 0.846 | 33 | 13 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.751 | 0.699 | 0.808 | 0.871 | 0.892 | 0.781 | 0.858 | 86 | 41 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi là `0.920`, thấp hơn IDF1 là `0.961`. Điều này cho thấy annotation vừa giữ identity khá nhất quán, vừa có một số lỗi về detection/localization như `33` FP và `13` FN; số IDSW là `0` nên không có lỗi đổi ID được ghi nhận. Nếu MOTA cao nhưng IDF1 thấp, model hoặc annotation có thể vẫn phát hiện đúng nhiều bbox nhưng giữ identity không nhất quán, chẳng hạn một xe bị đổi ID hoặc bị tách thành nhiều track. MOTA không phạt nặng lỗi ID vì mỗi ID switch chỉ bị tính như một lỗi trong công thức MOTA, trong khi IDF1 đánh giá việc ghép đúng identity trên toàn bộ quãng đời của track. Vì vậy một track bị đổi ID trong nhiều frame có thể làm IDF1 giảm đáng kể nhưng chỉ làm MOTA giảm ít.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID có kết quả association tốt hơn ByteTrack: IDF1 tăng từ `0.875` lên `0.900` và AssA tăng từ `0.776` lên `0.820`. Tuy nhiên, IDSW của hai phương pháp đều là `2`, nên ReID chưa làm giảm số lần đổi ID theo metric này. Điều đó cho thấy treatment có thể giữ identity và ghép track nhất quán hơn trên nhiều frame, nhưng vẫn còn một số tình huống tracker đổi ID.

Hiện chưa thể kết luận một frame sequence cụ thể vì các file evidence `outputs/model_bytetrack_clip_01.txt`, `outputs/model_reid_clip_01.txt` và JSON chẩn đoán tương ứng chưa có trong workspace. Cần bổ sung một sequence gồm frame, ID của gold, ID ByteTrack và ID ReID sau khi chạy model; không nên tự tạo số frame/ID khi chưa có evidence. Kết quả này cũng không cô lập causal effect của ReID, vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau, không chỉ khác mỗi việc bật ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với ByteTrack, BoT-SORT + ReID có DetA tăng từ `0.649` lên `0.711`, FN giảm từ `54` xuống `26` nhưng FP tăng nhẹ từ `88` lên `91`. Như vậy treatment bắt được nhiều xe bị bỏ sót hơn, nhưng cũng tạo thêm một số bbox dương tính giả. Phần lỗi còn lại liên quan đến detector vẫn đáng chú ý vì FP còn cao và số bbox phát hiện không khớp hoàn toàn với gold. Tuy nhiên, association cũng là một phần của vấn đề: AssA tăng từ `0.776` lên `0.820`, IDF1 tăng từ `0.875` lên `0.900`, nhưng IDSW vẫn là `2` ở cả hai tracker. Vì vậy ReID cải thiện việc ghép và giữ identity, còn các lỗi FP/FN còn lại chủ yếu cần xem lại khả năng phát hiện, ngưỡng confidence và bbox; không thể quy toàn bộ lỗi còn lại chỉ cho detector hoặc chỉ cho association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Chưa thể xác định một frame và ID cụ thể vì các file output model `outputs/model_reid_clip_01.txt` và evidence theo từng frame chưa có trong workspace. Do đó, tôi không kết luận ReID sai ở frame nào khi chưa có bbox/ID để đối chiếu với annotation và gold. Cần chạy lại model, sau đó chọn một frame mà gold và annotation của tôi có bbox đúng nhưng output ReID bị bỏ sót, có FP hoặc gán sai ID; khi đó ghi rõ frame, ID và loại lỗi vào mục này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Chưa có frame và ID cụ thể để kết luận ReID làm annotation cần sửa, vì file output ReID và evidence trực quan theo frame chưa được tạo trong workspace. Tôi không dùng riêng output của model để thay đổi annotation; nếu model khác annotation, cần đối chiếu frame đó với ảnh gốc, gold và quy tắc bbox trước khi quyết định. Với evidence hiện có, các chỉ số chỉ cho thấy ReID là một tín hiệu để xem lại các frame nghi ngờ, chưa đủ để chứng minh annotation sai hoặc model đúng.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` các quy tắc rõ hơn về frame bắt đầu/kết thúc của track: chỉ bắt đầu khi xác định chắc đó là xe bốn bánh, bấm `outside` ngay khi xe rời khung, và không giữ bbox dự đoán ở các frame không còn thấy xe. Tôi cũng sẽ ghi rõ phải thêm keyframe ở giữa đoạn xe đổi hướng, bị che hoặc thay đổi kích thước nhanh; sau đó tua lại giữa các keyframe để kiểm tra interpolation. Ngoài ra, guideline cần nhắc thao tác click trực tiếp vào bbox trước khi chỉnh và phải bấm Save trên CVAT rồi reload để xác nhận dữ liệu còn nguyên.

Trong quy trình, tôi sẽ làm xong từng track rồi mới chuyển sang track khác, kiểm tra ngay frame đầu/cuối và các frame giữa trước khi tạo track tiếp theo. Sau khi hoàn tất, tôi sẽ thực hiện ba lượt QC: kiểm tra ID, kiểm tra đầu/cuối track và kiểm tra giữa các keyframe; tiếp theo chạy `check_mot_labels.py`, export MOT 1.1, kiểm tra số ID và khóa pre-gold trước khi mở reference. Khi có lỗi từ gold, tôi sẽ đối chiếu frame với ảnh gốc, sửa trên CVAT, export lại rồi mới chạy đánh giá sau rework.

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

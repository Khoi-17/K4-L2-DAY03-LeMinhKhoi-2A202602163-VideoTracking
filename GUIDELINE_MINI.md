# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `...`
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

Bổ sung của nhóm (nếu có): Không có; áp dụng các quy tắc mặc định của lab.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Cùng một xe vẫn giữ identity khi thời gian che ngắn. |
| Xe bị che lâu hơn ngưỡng trên | mở track mới sau khi xác định lại đúng xe | Sau hơn 2 giây, không đủ chắc chắn để nối identity cũ. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Khi đã ra khỏi khung, không đủ bằng chứng chắc chắn để nối lại identity cũ. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo từng xe; thêm keyframe dày trước, trong và sau vùng cắt nhau | Giảm nguy cơ interpolation trôi hoặc đổi ID giữa hai xe. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định chắc là xe bốn bánh; nếu chưa chắc thì chờ frame rõ hơn |
| Xe đang đỗ, không di chuyển | vẫn gán và giữ track trong toàn bộ thời gian xe còn trong khung |
| Keyframe đặt dày ở đâu | đặt dày khi xe đổi hướng, phanh, bị che, thay đổi kích thước nhanh hoặc bbox bị lệch; tua lại giữa các keyframe để kiểm tra interpolation |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 82–100 / ID 6`
- Tình huống: ID 6 có bbox trong khi xe tham chiếu chưa xuất hiện ở đoạn này.
- Quyết định: đặt `outside` hoặc xóa phần bbox thừa; chỉ bắt đầu track khi xác định xe thực sự xuất hiện.
- Lý do: tránh bbox treo làm tăng FP và làm sai thời gian sống của track.

### Ca 2
- Clip / frame / ID: `clip_01 / 82, 83, 87 / ID 5`
- Tình huống: bbox ID 5 bị lệch so với phần xe nhìn thấy trong các frame chuyển động.
- Quyết định: click trực tiếp vào bbox, chỉnh sát xe và thêm keyframe nếu interpolation vẫn lệch.
- Lý do: bbox chỉ ôm phần nhìn thấy, không lấy nền hoặc phần xe không quan sát được.

### Ca 3
- Clip / frame / ID: `clip_01 / 103, 104, 111, 112 / ID 6`
- Tình huống: bbox ID 6 trôi khi xe thay đổi vị trí/kích thước.
- Quyết định: chỉnh lại bbox ở từng vùng chuyển động, thêm keyframe và kiểm tra các frame nằm giữa.
- Lý do: keyframe thưa có thể làm CVAT nội suy bbox lệch khỏi xe.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Phải kiểm tra frame đầu và cuối của từng track; khi xe rời khung phải bấm `outside`, không để bbox tiếp tục nội suy trên nền.
- Phải kiểm tra giữa các keyframe và thêm keyframe tại vùng chuyển động nhanh; không chỉ kiểm tra bbox ở hai đầu đoạn.
- Khi chỉnh bbox trên CVAT phải click trực tiếp vào bbox trước, sau đó Save và reload để xác nhận thay đổi đã được lưu.
- Sau khi export lại MOT 1.1, phải chạy validator và đánh giá với gold; không sửa trực tiếp file pre-gold.

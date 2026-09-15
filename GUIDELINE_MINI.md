# Mini annotation guideline — Ngày 3 (tracking)

Tên: `NGUYỄN ĐỨC TÙNG (CÁ NHÂN)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: `vehicle` — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của tôi | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới 25 frame (~2 giây @ 12.5 fps), theo mặc định của lab | Che ngắn hạn không đủ để xe đó biến mất khỏi hiện trường thực tế, giữ ID giúp phản ánh đúng identity liên tục |
| Xe bị che lâu hơn ngưỡng trên | Vẫn cân nhắc giữ ID nếu vẫn quan sát được vị trí/hướng di chuyển hợp lý trước và sau khi che, dựa trên đối chiếu bằng mắt qua `visualize_tracks.py` | Che lâu vẫn có thể là cùng một xe nếu bối cảnh (làn đường, tốc độ, hướng đi) khớp logic, không nên cứng nhắc tách ID chỉ vì vượt ngưỡng |
| Xe rời khung hình rồi quay lại | Mặc định: track mới | Không đủ cơ sở đảm bảo đây là cùng một xe quay lại, tránh gán nhầm identity |
| Hai xe cắt nhau / chồng lên nhau | Theo dõi kỹ hướng di chuyển và tốc độ trước khi hai xe chồng nhau để giữ đúng ID cho từng xe sau khi tách ra, chỉnh thủ công từng frame ở đoạn chồng lấn | Tự động interpolate dễ nhầm lẫn ID giữa hai xe có quỹ đạo gần nhau |

## 3. Luật bbox

| Tình huống | Luật của tôi |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm phần nhìn thấy được |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định chắc chắn là xe bốn bánh (không đoán từ hình dạng mờ) |
| Xe đang đỗ, không di chuyển | Vẫn giữ track xuyên suốt với bbox gần như không đổi vị trí, xác nhận bằng quan sát trực tiếp là xe không di chuyển chứ không phải lỗi quên chỉnh box |
| Keyframe đặt dày ở đâu | Đặt dày ở các đoạn xe đổi hướng, đổi tốc độ, hoặc bắt đầu/kết thúc bị che, để interpolation không bị lệch |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1 (từ clip_02)
- Clip / frame / ID: `clip_02`, frame 1–3, ID 2
- Tình huống: track chỉ tồn tại đúng 3 frame, validator cảnh báo nghi vẽ nhầm
- Quyết định: giữ nguyên track, không xóa
- Lý do: xem lại bằng `visualize_tracks.py`, xác nhận đây là xe thật đi qua rất ngắn (ở rìa khung hoặc bị che gần hết), không phải lỗi thao tác

### Ca 2 (từ clip_01)
- Clip / frame / ID: `clip_01`, frame 1–189, ID 3
- Tình huống: box đứng yên gần như toàn bộ clip, validator nghi ngờ có thể do quên di chuyển bbox
- Quyết định: giữ nguyên, không sửa
- Lý do: xe thật sự đang đỗ/dừng tại chỗ trong suốt clip, không di chuyển, xác nhận qua quan sát trực tiếp

### Ca 3 (từ kết quả evaluate vs gold)
- Clip / frame / ID: `clip_01`, frame 79–100, ID 6 (và tương tự các đoạn thừa ở ID 4, 5, 7, 8)
- Tình huống: evaluate cho thấy track của tôi có đoạn "thừa" so với gold — có bbox ở đoạn mà gold ghi nhận track tham chiếu chưa xuất hiện
- Quyết định: giữ nguyên theo phán đoán của mình, không chỉnh sửa theo gold
- Lý do: tôi khẳng định ở đoạn đó có thêm phương tiện xuất hiện đồng thời với chiếc xe buýt, quan sát bằng mắt cho thấy có xe thật ở vị trí đó sớm hơn thời điểm gold ghi nhận

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Cần làm rõ hơn ngưỡng số frame che khuất cụ thể cho từng loại xe (xe lớn như buýt/tải có thể cần ngưỡng khác xe con do tốc độ và kích thước khác nhau).
- Cần thống nhất quy trình xác minh khi có sự khác biệt giữa annotation cá nhân và gold — nên có bước đối chiếu thêm bằng công cụ trực quan trước khi khẳng định annotation của mình đúng, để giảm rủi ro chủ quan.

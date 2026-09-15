# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Ngo Minh Tuan — Individual`
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

Bổ sung của nhóm (nếu có): `Chỉ gán vehicle bốn bánh thật trong cảnh; không gán người, xe đạp, xe máy/mô tô, hình xe trong quảng cáo, gương, bóng nước hoặc vật thể tĩnh không phải xe.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Vẫn đủ bằng chứng về quỹ đạo và hình ảnh để xác định đó là cùng một vehicle |
| Xe bị che lâu hơn ngưỡng trên | mở track mới khi xe xuất hiện lại nếu identity không còn chắc chắn | Tránh nối nhầm hai xe khác nhau |
| Xe rời khung hình rồi quay lại | **track mới** | Không còn quan sát liên tục sau khi xe đã rời hoàn toàn khỏi khung |
| Hai xe cắt nhau / chồng lên nhau | giữ ID dựa theo quỹ đạo trước và sau giao nhau; thêm keyframe quanh đoạn giao nhau; không đổi ID chỉ vì hai bbox chồng lên nhau | Quỹ đạo liên tục đáng tin cậy hơn độ chồng lấp tức thời |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chỉ bao phần nhìn thấy, chạm đúng rìa ảnh và không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox chỉ ôm phần **nhìn thấy được**; dùng thuộc tính Occluded khi phù hợp |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định đủ chắc chắn là xe bốn bánh; không đoán khi chỉ thấy vài pixel mơ hồ |
| Xe đang đỗ, không di chuyển | vẫn gán nếu là xe thật trong cảnh và còn nhìn thấy; đứng yên không làm mất nhãn vehicle |
| Keyframe đặt dày ở đâu | khi xe đổi hướng hoặc tốc độ, gần mép ảnh, bị che, giao nhau với xe khác, hoặc khi interpolation bắt đầu làm bbox trôi |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 79–100 / Annotation ID 6`
- Tình huống: track bắt đầu sớm hơn reference.
- Quyết định: bổ sung guideline chỉ bắt đầu track khi vehicle đủ rõ.
- Lý do: đây là finding phát hiện sau khi đối chiếu gold; final annotation không được rework và vẫn giữ nguyên pre-gold.

### Ca 2
- Clip / frame / ID: `clip_01 / 149–151 / Annotation ID 4`
- Tình huống: bbox còn sau khi reference đã rời khung.
- Quyết định: bổ sung bước kiểm tra frame cuối và dùng Outside khi xe biến mất.
- Lý do: finding được phát hiện sau gold nhưng chưa rework; final annotation vẫn giữ nguyên pre-gold.

### Ca 3
- Clip / frame / ID: `clip_01 / 84 / Annotation ID 5`
- Tình huống: bbox giữa đoạn interpolation bị lệch, IoU khoảng `0.506` so với reference.
- Quyết định: bổ sung keyframe ở đoạn interpolation dài hoặc khi bbox bắt đầu trôi.
- Lý do: finding được phát hiện sau gold nhưng chưa rework; final annotation vẫn giữ nguyên pre-gold.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Bài được thực hiện cá nhân nên không có peer review. Sau khi đọc gold/diagnostics, bổ sung bước kiểm tra frame đầu và frame cuối của từng track kỹ hơn.
- Đặt keyframe dày hơn ở occlusion, giao nhau, mép ảnh và đoạn interpolation dễ làm bbox trôi.
- Các cập nhật guideline này được rút ra từ gold/diagnostics; final annotation không được rework và vẫn giống hoàn toàn bản pre-gold.

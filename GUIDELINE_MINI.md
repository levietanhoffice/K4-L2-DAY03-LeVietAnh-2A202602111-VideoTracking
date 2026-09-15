# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Việt Anh`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps); bật thuộc tính `Occluded` tại các frame bị che. | Quỹ đạo và vận tốc xe vẫn mang tính liên tục qua điểm mù; giữ ID giúp mô hình học đúng liên kết appearance/motion mà không làm vỡ track. |
| Xe bị che lâu hơn ngưỡng trên | Bấm `Outside` để kết thúc track cũ; khi xe lộ diện trở lại sau hơn 25 frame thì tạo **track mới với ID mới**. | Thời gian che khuất quá dài làm mất tính liên tục động học, khó đảm bảo chắc chắn là cùng một xe nếu không thấy biển số; tách ID để tránh rủi ro gán nhầm xe khác gây nhảy ID  nặng. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** với ID mới. | Tuân thủ chuẩn: đối tượng đã biến mất hoàn toàn khỏi trường nhìn rồi xuất hiện lại được tính là một trajectory mới. |
| Hai xe cắt nhau / chồng lên nhau | Duy trì đúng ID của từng xe trước, trong và sau khi giao cắt. Xe ở lớp sau (bị che) bật thuộc tính `Occluded` và giữ kích thước nội suy theo quán tính. | Tránh hiện tượng hoán đổi ID giữa 2 xe khi tách rời nhau sau điểm giao cắt. |
## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm sát đúng mép ảnh, cắt thẳng theo đường biên, tuyệt đối không vẽ ước lượng phần thân xe nằm ngoài khung hình. |
| Xe bị xe khác che một phần | Bbox ôm sát phần **nhìn thấy được (visible box)**, không vẽ lấn sang thân xe phía trước; đồng thời bật cờ `Occluded`. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên thấy xe và kích thước bbox tối thiểu đạt từ **10x10 pixel** trở lên. |
| Xe đang đỗ, không di chuyển | Vẫn gán nhãn và duy trì 1 ID duy nhất trong suốt thời gian xe xuất hiện; chỉ cần đặt 2 keyframe (frame đầu tiên và frame cuối cùng trước khi rời khung hình/hết clip). |
| Keyframe đặt dày ở đâu | Cắm keyframe dày (khoảng cách **3–5 frame/keyframe**) tại các đoạn xe giảm tốc/phanh gấp, chuyển làn, hoặc rẽ góc cua ngã tư để chống hiện tượng bbox bị trôi lệch tâm (interpolation drift). |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- **Clip / frame / ID:** `clip_01 / frame 74–100 / ID 5 và ID 6`
- **Tình huống:** Xe xuất hiện từ xa ở góc đường rẽ vào, ban đầu chỉ là vệt mờ nhỏ sát mép tán cây. Người gán nhãn phân vân có nên track ngay từ frame 74 (ID 5) và 79 (ID 6) hay không.
- **Quyết định:** Không track quá sớm; chỉ bắt đầu cắm keyframe tạo track khi xe đã tiến vào làn chính và thấy rõ cabin (từ frame 79 với ID 5 và frame 101 với ID 6).
- **Lý do:** Gán quá sớm khi đối tượng chưa đạt ngưỡng kích thước nhận dạng sẽ sinh ra nhiều False Positive (bbox treo/thừa), làm lệch chuẩn với tập nhãn Gold.

### Ca 2
- **Clip / frame / ID:** `clip_01 / frame 105–115 / ID 6`
- **Tình huống:** Xe ID 6 rẽ giữa ngã tư và bị xe đi ngược chiều che khuất hơn 50% thân xe trong khoảng 8 frame.
- **Quyết định:** Giữ nguyên ID 6 xuyên suốt, co viền bbox ôm phần thân xe còn nhìn thấy, bật thuộc tính `Occluded` trên CVAT và không bấm `Outside`.
- **Lý do:** Thời gian che khuất chỉ diễn ra trong 8 frame (< 25 frame), hướng di chuyển của xe có thể suy luận chính xác, giữ nguyên ID giúp duy trì chỉ số liên kết AssA cao và đạt 0 ID Switch.

### Ca 3
- **Clip / frame / ID:** `clip_01 / frame 149–151 (ID 4) và frame 169–171 (ID 8)`
- **Tình huống:** Xe di chuyển ra khỏi mép dưới khung hình; ở 2-3 frame cuối xe chỉ còn một phần cản sau rồi biến mất hoàn toàn.
- **Quyết định:** Bấm phím tắt `O` (Outside) ngay tại frame đầu tiên xe khuất hẳn (frame 149 cho ID 4; frame 169 cho ID 8) để kết thúc track lập tức.
- **Lý do:** Nếu không bấm `Outside`, CVAT sẽ tự động nội suy tuyến tính kéo bbox lơ lửng ngoài khung hình ở các frame sau, tạo ra lỗi bbox treo (bị phạt trừ điểm nặng ở MOTA).

---

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy định dứt khoát về frame kết thúc (`Outside`):** Bắt buộc phải bấm phím tắt `O` (Outside) ngay tại frame đối tượng vừa ra khỏi khung hình hoặc bị che khuất hoàn toàn, tuyệt đối không để bbox trôi tự do theo keyframe cuối.
- **Quy tắc viền bbox khi có bóng đổ (Shadow):** Bbox chỉ bao quanh phần vỏ kim loại cứng và điểm tiếp giáp của bánh xe với mặt đường; không bao gồm vệt bóng đen của xe in trên mặt đường và không kéo dài ra phần mờ của gương chiếu hậu khi xe chạy nhanh.
- **Quy chuẩn mật độ keyframe:** Tại các phân đoạn xe chuyển hướng (rẽ ngã tư) hoặc thay đổi tốc độ, khoảng cách tối đa giữa 2 keyframe không được vượt quá 5 frame nhằm đảm bảo điểm IoU luôn đạt 0.70, hạn chế tối đa lỗi bbox trôi (drift).
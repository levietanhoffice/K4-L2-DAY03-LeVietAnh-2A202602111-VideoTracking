# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Việt Anh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT  |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `12` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất một phần (Occlusion) bởi xe khác:** Xử lý bằng cách duy trì kích thước phán đoán của bounding box theo quỹ đạo chuyển động đều, tích chọn thuộc tính `Occluded` trên CVAT và giữ nguyên `track_id` để không bị đứt track khi xe lộ diện trở lại.
2. **Xác định thời điểm đối tượng ra/vào rìa khung hình:** Xe xuất hiện dần dần hoặc khuất dần ở mép ảnh; xử lý bằng cách bắt đầu vẽ khi nhìn rõ mũi xe và bấm phím tắt bật tính năng `Outside` ngay tại frame đầu tiên xe khuất hẳn để tránh tạo ra các bbox treo (treo lơ lửng ngoài vùng xe chạy).
3. **Bbox bị trôi khi xe đổi hướng hoặc giảm tốc:** Xử lý bằng cách chèn thêm các keyframe trung gian tại điểm xe bắt đầu đổi hướng hoặc hãm phanh.

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
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `d60d630524560a15380ce99b9a7a106ed126a7efd9fc0a38dd50dcfdb3d565fb` |
| Thời điểm khóa | `4h47` |
| Số row / frame / track trước khi mở reference | `620 row / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.756 | 0.739 | 0.775 | 0.852 | 0.919 | 0.831 | 0.841 | 72 | 25 | 0 |
| Sau rework | 0.768 | 0.751 | 0.786 | 0.860 | 0.928 | 0.842 | 0.850 | 65 | 21 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có (ĐẠT)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa / xuất hiện sớm | 79–100 | ID 6 | Bỏ 22 bbox thừa xuất hiện trước khi xe vào vùng quan sát chuẩn của gold |
| Bbox thừa / xuất hiện sớm | 74–78 | ID 5 | Điều chỉnh frame bắt đầu track lùi lại đúng mốc frame 79 |
| Bbox treo / sót sau khi rời khung | 149–151 | ID 4 | Bật thuộc tính `Outside` ngay tại frame 149 khi xe vừa khuất |
| Bbox treo / sót sau khi rời khung | 169–171 | ID 8 | Bật thuộc tính `Outside` tại frame 169 |
| Bbox trôi (IoU thấp ~0.51) | 84–90 | ID 5 | Thêm 2 keyframe tại frame 86 và 88, nắn lại bbox ôm sát thân xe |
| Bbox trôi (IoU thấp ~0.52) | 102–105 | ID 6 | Thêm keyframe tại frame 103 để định hình lại góc nghiêng của xe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) & `botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes | `0.25` / `0.70` / `960` / `[2, 5, 7]` (car, bus, truck) |
| device | `0` (GPU CUDA) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.756 | 0.739 | 0.775 | 0.852 | 0.919 | 0.831 | 0.841 | 72 | 25 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.707 | 0.653 | 0.767 | 0.860 | 0.859 | 0.718 | 0.847 | 95 | 77 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Trong bản ban_vs_gold, MOTA (0.831) thấp hơn IDF1 (0.919). Nếu một kết quả có MOTA cao nhưng IDF1 thấp, điều đó nghĩa là detector phát hiện đối tượng từng frame rất tốt (ít FP, FN) nhưng tracker liên tục bị nhảy ID (nhiều ID Switch hoặc vỡ track). MOTA không phạt nặng lỗi ID vì công thức MOTA chỉ trừ 1 điểm phạt tại đúng frame xảy ra ID Switch; ở các frame tiếp theo, nếu bbox vẫn khớp vị trí thì nó vẫn được tính là True Positive dưới ID mới. Ngược lại, IDF1 tính toán F1-score trên toàn bộ chiều dài quỹ đạo (ID trajectory match); nếu một track bị đổi ID giữa chừng, toàn bộ nửa sau của track đó sẽ bị coi là ID sai, khiến IDF1 tụt rất sâu.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`So với ByteTrack control, BoT-SORT + ReID treatment cải thiện rõ rệt: AssA tăng từ 0.776 lên 0.820 (+0.044), IDF1 tăng từ 0.875 lên 0.900 (+0.025), trong khi IDSW giữ nguyên ở mức 2 lần. 
Cụ thể ở chuỗi frame 130–170 đối với track gold 8: ByteTrack chỉ bao phủ được 20/33 frame (61%) và làm mất dấu khi xe bị biến dạng góc nhìn/che khuất nhẹ; trong khi BoT-SORT + ReID bám đuổi trọn vẹn track 8 nhờ vector đặc trưng diện mạo (appearance embedding) giúp kết nối lại tracklet sau khi bộ lọc Kalman bị mất đà. 
Cần lưu ý: Thí nghiệm này là một so sánh cấp hệ thống (system comparison), KHÔNG cô lập được hiệu ứng nhân quả (causal effect) riêng biệt của ReID, bởi vì ByteTrack và BoT-SORT khác nhau về cấu trúc pipeline (BoT-SORT tích hợp Camera Motion Compensation - CMC, tinh chỉnh ma trận Kalman và ngưỡng kết hợp khác với ByteTrack).`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`Khi chuyển từ ByteTrack sang BoT-SORT + ReID: DetA tăng từ 0.649 lên 0.711, số lượng bỏ sót (FN) giảm mạnh hơn một nửa từ 54 xuống 26, trong khi FP tăng nhẹ từ 88 lên 91. 
Lỗi lớn nhất còn lại chủ yếu nằm ở DETECTOR chứ không phải association:
- Mô hình sinh ra khoảng 91 FP chủ yếu do detector nhận diện các xe ở quá xa, mép ảnh mờ (như track ID 7 tồn tại 43 frame, ID 27 tồn tại 16 frame, ID 38 tồn tại 16 frame) mà bản gold quy chuẩn không gán nhãn.
- Khâu association hoạt động rất tốt với AssA đạt 0.820 và chỉ có 2 lần IDSW trên toàn clip.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Tại frame 107–113 đối với xe rẽ ở giữa ngã tư (track gold 6): Bản gán nhãn tay của bạn giữ vững ID 6 xuyên suốt quãng đời (79 frame liên tục). Ngược lại, ReID treatment bị tách track và nhảy ID liên tiếp (frame 107 từ ID 24 sang ID 28, rồi frame 110 nhảy sang ID 31). Nguyên nhân là do xe bị che khuất tạm thời bởi phương tiện khác làm detector bị đứt quãng; khi xe lộ ra, góc chiếu sáng và hình dạng thay đổi khiến ReID embedding không đủ tự tin để nối lại ID cũ mà tạo ID mới.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Tại cụm frame 108, 109, 118 (danh sách worst frames): Báo cáo chỉ ra "chỉ model có 3, chỉ bạn có 2". Khi kiểm tra lại frame 108–109, ReID phát hiện được một phương tiện nhỏ ở làn đường đối diện phía xa (ID 27) mà mắt thường khi gán nhãn tua nhanh đã bỏ sót vì kích thước vật thể nhỏ hơn 20 pixel. Đây là trường hợp model giúp người gán nhãn nhận ra thiếu sót ở vùng rìa xa. Tuy nhiên, tại cùng frame đó model cũng bắt nhầm một biển báo tĩnh thành xe (ID 7 kéo dài từ frame 16–116 không di chuyển), ở trường hợp này bằng chứng cho thấy model đã bị False Positive.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`- Sửa đổi trong GUIDELINE_MINI.md:
  1. Định nghĩa rõ tiêu chuẩn kích thước tối thiểu (ví dụ: bbox có chiều dài tối thiểu > 25px hoặc thấy rõ > 20% thân xe mới gán nhãn) để thống nhất việc có theo dõi các xe ở viền xa hay không.
  2. Bổ sung quy tắc ranh giới chặt chẽ: bbox chỉ bao quanh phần thân/vỏ cứng của xe, tuyệt đối không tính bóng xe trên mặt đường và không kéo dài ra gương chiếu hậu nếu bị nhòe.
  3. Bổ sung hướng dẫn bắt buộc: nếu xe giảm tốc độ hoặc rẽ hướng trên 30 độ, khoảng cách giữa 2 keyframe không được vượt quá 5 frame để hạn chế lỗi trôi bbox.
- Thay đổi quy trình làm việc:
  1. Tuân thủ nghiêm ngặt quy trình kiểm tra 3 lượt (Lượt 1 kiểm tra tính liên tục của ID; Lượt 2 kiểm tra toggle Outside ở frame đầu/cuối; Lượt 3 tua chậm để nắn bbox trôi).
  2. Tận dụng tối đa phím tắt CVAT ('O' để toggle Outside, 'K' để set keyframe) thay vì thao tác chuột thủ công.`

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)

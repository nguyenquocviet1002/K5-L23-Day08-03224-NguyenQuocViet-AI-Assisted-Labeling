# Báo cáo Day 08 — 03224 - Nguyễn Quốc Việt

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / car / bus / truck | YOLO26m | 0.3 (mặc định) | `r01`, `r05`: Các lớp này trong ảnh gold có ít hơn 5 mẫu hoặc không có ngưỡng nào $\le 0.5$ đạt Recall 90% (theo `threshold_sweep.md`). Dùng ngưỡng an toàn 0.3. |
| motorbike | YOLO26m | 0.1 | `r01`, `r05`: Tại ngưỡng 0.1 đạt Recall 93% ($\ge 90\%$), Precision 76% trên 14 vật gold (`threshold_sweep.md`). |
| three_wheeler | YOLOE-26s | 0.15 | `r03`, `r06`: Open-vocabulary với prompt "three_wheeler" đạt Recall 100%, Precision 78% trên 7 vật gold tại ngưỡng 0.15. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split batch,gold --out work/prelabel/merged_review/pre.coco.json
```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 base** (`sam2.1_b.pt`, `r04`) vì đây là phiên bản cân bằng xuất sắc nhất giữa chất lượng mask, độ bám biên chi tiết của các nan hoa xe / mui xe phức tạp và tốc độ suy luận nhanh trên GPU Google Colab T4 (~1.9s/ảnh).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`): Lỗi phân loại sai do tập dữ liệu huấn luyện COCO không có khái niệm xe ba bánh đặc thù Châu Á (`three_wheeler`), dẫn tới YOLO26 luôn gán tuk-tuk thành `car` hoặc `truck` với độ tin cậy rất cao (conf > 0.8). Sửa bằng cách dùng model open-vocabulary YOLOE-26s nhận diện riêng lớp `three_wheeler` và nạp đè vào qua `merge_prelabels.py`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở `img_013.jpg`, xe tuk-tuk bị YOLO26 dự đoán nhầm thành `car` với điểm tin cậy cao 0.82.
  - Điểm thấp mà đúng: Ở `img_002.jpg`, xe xích lô được YOLOE phát hiện với conf chỉ 0.18 nhưng vị trí box hoàn toàn khớp với thực tế.
- Giới hạn: Chọn ngưỡng trên chính 4 ảnh gold (in-sample) nên số đo Precision / Recall của máy trên gold mang tính lạc quan; tập mẫu 4 ảnh quá nhỏ để khái quát hóa toàn bộ phân phối giao thông.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 571 | 571/622 = 91.8% | 622 box máy |
| edited (sửa box/nhãn) | 8 | 8/622 = 1.3% | 622 box máy |
| deleted (xoá box thừa) | 43 | 43/622 = 6.9% | 622 box máy |
| added (vẽ thêm box sót) | 25 | 25/604 = 4.1% | 604 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp **`three_wheeler`** là tệ nhất: máy tạo ra 22 box bị xoá (`deleted`) và 7 box phải sửa (`edited`), chỉ có 12 box được `accepted`. Nguyên nhân do YOLOE nhạy cảm với các khung bảng hiệu hoặc xe tải nhỏ ở góc hẹp (như trong `img_005.jpg`), đồng thời bị chồng lấn với box sai của YOLO26.
- Trong số deleted + added có 8 trường hợp là đổi lớp (máy định vị đúng nhưng sai nhãn), do đó số lượng máy bỏ sót thực tế là $25 - 8 = 17$ đối tượng.
- So với gold: Precision của máy đạt **68.7%** (46/67, KTC 95%: 57–78%), Recall đạt **82.1%** (46/56 đối tượng gold, KTC 95%: 70–90%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **6/12 = 50.0%** (KTC 95%: 25–75%).
  - Bắt được theo từng loại: `missing` (2/3), `extra` (2/3), `wrong_class` (2/3), `loose` (0/3).
  - Danh sách bẫy lọt: `img_029.jpg` (loose), `img_031.jpg` (loose), `img_027.jpg` (loose), `img_028.jpg` (extra), `img_006.jpg` (wrong_class), `img_004.jpg` (missing).
- Vì sao lọt bẫy (đã bỏ bước nào của checklist)?
  - Toàn bộ 3/3 bẫy `loose` (box bị kéo lỏng ở `img_027`, `img_029`, `img_031`) đều bị lọt do người duyệt bỏ qua Bước 4 (kiểm tra độ ôm khít của box). Khi thấy box đã nằm đúng vị trí người đi bộ và nhãn đúng là `person`, mắt có xu hướng bỏ qua việc zoom sát để kiểm tra mép biên đỉnh đầu và chân.
  - Bẫy `extra` ở `img_028.jpg` lọt do box nằm ở vùng hậu cảnh tối khuất. Bước 0 (quét ảnh trước khi mở nhãn) chưa được thực hiện triệt để theo từng góc phần tư.
- Phân tích mối quan hệ: **Accepted rất cao (91.8%) nhưng bắt bẫy thấp (50.0%)**:
  - Đây là biểu hiện rõ rệt của **Thiên kiến tự động hóa (Automation Bias)**. Khi thấy đa số nhãn máy gợi ý trông có vẻ hợp lý, người duyệt chuyển sang trạng thái "quét nhanh và chấp nhận" (rubber-stamping) thay vì thực sự kiểm tra độc lập từng đối tượng.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`):
  - Gán tay thuần túy: **3.75 phút/ảnh** (15 phút / 4 ảnh gold).
  - Duyệt có pre-label: **1.25 phút/ảnh** (30 phút / 24 ảnh batch).
  - Tốc độ hoàn thành tăng gấp 3 lần (tiết kiệm 66.7% thời gian làm việc).
  - Thiên lệch: Người duyệt đã làm quen với bối cảnh ảnh khi gán 4 ảnh gold trước đó, tuy nhiên mức tiết kiệm thời gian vẫn là vượt trội so với vẽ thủ công từ đầu.
- Kết luận: **CÓ ĐÁNG DÙNG**, nhưng chỉ đáng tin cậy cao cho các lớp phổ biến (`person`, `motorbike`, `car`). Đối với các lớp khó như `three_wheeler` hoặc đối tượng che khuất, người duyệt phải duy trì sự tập trung cao độ và không được chủ quan.
- **Ba quy tắc giảm thiên kiến của bạn:**
  1. *Quy tắc quét mù 15 giây (Step 0):* Với mỗi ảnh mới, luôn ấn phím ẩn toàn bộ nhãn máy (`H`), tự dùng mắt quét 4 góc phần tư trong 15 giây để ước lượng số lượng người và phương tiện trước khi bật hiển thị box máy.
  2. *Quy tắc phóng to 150% kiểm biên (Anti-loose):* Bắt buộc zoom 150% vào ít nhất 3 đối tượng bất kỳ trên mỗi ảnh để soi mép biên (chân người, bánh xe), triệt tiêu bẫy box lỏng.
  3. *Quy tắc nghỉ mắt ngắt quãng (Pomodoro):* Cứ sau 8 ảnh duyệt liên tục (khoảng 10-12 phút), bắt buộc dừng lại 2 phút không nhìn màn hình để phục hồi độ tinh tường thị giác, chống mỏi mắt gây giảm khả năng phát hiện lỗi.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái; bao trọn khung xe tuk-tuk | Click âm ở vị trí người lái giúp tách riêng thân xe thành công | `tuktuk_img013.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 (1 pos, 1 neg) | Tách riêng xe xích lô khỏi người đạp phía sau | Một phần nan hoa bánh xe phía trước bị lẹm nhẹ | `sam_img004_cyclo.jpg` |
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 (2 pos) | Mask ôm trọn phần mui và cabin xe | Vùng kính chắn gió bị rỗng do vật liệu phản quang trong suốt | `sam_img013_tuktuk.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc tách riêng người lái (`person`) và phương tiện (`three_wheeler`) hoạt động rất tốt khi có click âm (`--neg`). Tuy nhiên guideline thiếu hướng dẫn cụ thể về việc xử lý phần kính chắn gió nhìn xuyên thấu (có tính vào mask thân xe hay không).
- So sánh với Interactor SAM trong CVAT: Chạy script offline qua lệnh `sam2_points.py` giúp kiểm soát toạ độ điểm chính xác, tuy nhiên thao tác tương tác click trực tiếp trên giao diện CVAT AI Tools mang lại phản hồi trực quan nhanh hơn cho annotator.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** (trong `qc/calibration_decisions.csv`). Quan trọng nhất:
  - `CAL-01`: Quy định chuẩn hóa mọi xe ba bánh có mui, xích lô, ba gác về lớp `three_wheeler` để khắc phục triệt để lỗi phân loại của YOLO26.
  - `CAL-03`: Quy định bổ sung thao tác zoom 150% kiểm tra biên tiếp xúc của box để ngăn chặn lọt bẫy `loose`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung tiêu chuẩn phân loại xe đạp điện / xe máy điện (dựa vào biển số và bàn đạp cơ), quy định khối xe đỗ san sát và tách người điều khiển xích lô/tuk-tuk thành box riêng.
- Nguyên nhân gốc: Kết hợp giữa sự mơ hồ của guideline v1.0 (chưa định nghĩa xe điện và khối xe dày đặc) cùng với lỗi thiên vị dữ liệu của mô hình AI (domain gap).

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ca khó + 5 ngẫu nhiên, seed 42); gồm **122 đối tượng** (`qc_sample.csv`). Người kiểm: Nguyễn Quốc Việt (**hạn chế**: người kiểm cũng chính là người review, có thể chịu thiên kiến chủ quan tương tự).
- Lỗi theo **đối tượng**: **0/122 = 0.0%** · theo **ảnh**: **0/6 = 0.0%** (ngưỡng Release $\le 2\%$).
- Top lỗi: Không phát hiện lỗi nghiêm trọng trong 6 ảnh mẫu QC.
- **Giới hạn mẫu nhỏ**: Theo `qc_report_auto.md`, với 0 ảnh lỗi trên mẫu 6 ảnh, khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thực tế có cận trên lên tới **39.0%**. Do đó, kết quả 0% trên mẫu 6 ảnh hoàn toàn chưa đủ độ tin cậy thống kê để khẳng định tỷ lệ lỗi thực tế đạt chuẩn $\le 2\%$.
- Script kiểm tra: Tự động kiểm tra cú pháp, trường dữ liệu, format nhãn COCO; nhưng không thể tự động đánh giá độ khít của box hay phát hiện những vật thể bị cả người và máy cùng bỏ sót ở ngoài thực địa.

**Quyết định: REWORK**, ngày 2026-11-20.
- Lý do (theo cây quyết định Hold $\rightarrow$ Rework $\rightarrow$ Release):
  1. *Hold:* Không có vấn đề dữ liệu hỏng, các câu hỏi mở đã được chốt ở v1.1 $\rightarrow$ Không Hold.
  2. *Rework:* Mặc dù mẫu QC đạt 0% lỗi, nhưng **tỉ lệ bắt bẫy nhân tạo chỉ đạt 6/12 = 50.0% (< ngưỡng chuẩn 80%)**, đặc biệt bỏ lọt 100% bẫy box lỏng (`loose`). Điều này chứng minh lô duyệt 24 ảnh batch vẫn tiềm ẩn nhiều box chưa ôm sát vật thể do thiên kiến tự động hóa.
  3. Do đó, theo chuẩn quy trình chất lượng, quyết định chính xác là **REWORK** để rà soát lại lô dữ liệu.
- Việc cần làm tiếp khi Rework:
  - Tải lại bộ nhãn `batch_reviewed.json` vào CVAT.
  - Áp dụng triệt để quy tắc zoom 150% rà soát toàn bộ các box người đi bộ và xe máy ở 24 ảnh batch.
  - Sau khi nắn khít toàn bộ box lỏng, xuất nhãn cập nhật và kiểm định QC vòng 2.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng mô hình YOLO26m, YOLOE-26s, SAM 2.1 base trên môi trường Google Colab GPU T4 để sinh pre-label và mask; sử dụng AI Assistant để hỗ trợ rà soát mã nguồn kiểm thử và định dạng báo cáo.
- Gold có bị nhiễm không: **Hoàn toàn không**. 4 ảnh gold được gán tay thủ công 100% trong CVAT trước khi chạy bất kỳ model nào trên tập dữ liệu này.
- Môi trường: Máy Windows 11 x64, Google Colab (GPU NVIDIA Tesla T4), CVAT Docker v2.81.0. Không gặp trở ngại lớn về cài đặt nhờ tách biệt phần chạy model sang Colab.
- Dùng cache `data/cache/`: Không dùng file cache có sẵn; toàn bộ file pre-label thô đều được chạy trực tiếp trên Colab T4.
- Một điều bạn làm khác lần sau: Sẽ dành thời gian kiểm tra độ khít (tightness) của bounding box ngay từ đầu thay vì chỉ tập trung sửa nhãn lớp, nhằm tránh hoàn toàn việc để lọt các bẫy box lỏng (`loose`).

# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03224 - Nguyễn Quốc Việt

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân loại theo biển số và bàn đạp: Nếu có biển số đăng ký xe máy điện thì gán `motorbike`; nếu có bàn đạp cơ đạp chân và dáng xe đạp không biển số thì gán `bicycle`. | img_003.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Xe nhìn rõ $\ge 20\%$ thân xe gán riêng từng box; khối xe chen lấn bị che khuất $> 80\%$ không thể phân biệt ranh giới thì phủ `ignore_region`. | img_005.jpg | CAL-04 |
| 3 | Hàng nhô ra: xe đạp / ba gác chở tre, rổ nhô xa: box xe có gồm hàng không? | Box hình chữ nhật ôm trọn cả xe và hàng hoá gắn cố định trên xe; với mask (segmentation) thì chỉ phủ thân xe và giá đỡ hàng, không tô phần hàng cồng kềnh nhô ra ngoài. | img_002.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk và xích lô chở khách đường phố | YOLO26 gọi là `car` hoặc `truck` | Luôn chuyển về `three_wheeler` | Mục 2: Xe ba bánh (tuk-tuk, xích lô, xe lôi, ba gác) bất kể có mui che hay thùng sau đều thuộc lớp `three_wheeler`. |
| img_029.jpg | Box người đi bộ bị kéo lỏng (bẫy loose) | Bị lọt bẫy do mắt chỉ tập trung nhìn nhãn lớp | Kéo lại sát mép đỉnh đầu và gót chân người | Mục 4.1: Bổ sung bước phóng to (zoom 150%) kiểm tra mép biên của các box người và phương tiện. |
| img_004.jpg | Người lái xích lô ngồi phía sau khách | YOLOE gán box xe bao trùm cả người lái | Tách riêng box `person` cho người lái và box `three_wheeler` cho xe | Mục 5: Người điều khiển phương tiện luôn được gán một box `person` riêng biệt, không gộp vào xe. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | 1. Quy định rõ xe điện (biển số $\rightarrow$ `motorbike`, bàn đạp $\rightarrow$ `bicycle`).<br>2. Quy chuẩn hóa xe ba bánh Châu Á (tuk-tuk, xích lô $\rightarrow$ `three_wheeler`).<br>3. Quy định khối xe đỗ san sát và checklist kiểm tra box lỏng (zoom 150%). | Giải quyết câu hỏi mở v1.0 (CAL-02, CAL-04) và khắc phục lỗi phân loại của model YOLO26 cùng bẫy lọt (CAL-01, CAL-03). |

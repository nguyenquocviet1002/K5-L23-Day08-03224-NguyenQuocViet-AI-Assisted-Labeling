# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 622; đối tượng sau duyệt: 604; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 571 | 571/622 = 91.8% | 622 box máy |
| edited (sửa) | 8 | 8/622 = 1.3% | 622 box máy |
| deleted (xoá) | 43 | 43/622 = 6.9% | 622 box máy |
| added (vẽ thêm) | 25 | 25/604 = 4.1% | 604 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 17.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 264 | 0 | 6 | 7 |
| bicycle | 6 | 0 | 2 | 3 |
| motorbike | 230 | 1 | 9 | 3 |
| car | 52 | 0 | 2 | 1 |
| bus | 3 | 0 | 0 | 3 |
| truck | 4 | 0 | 2 | 5 |
| three_wheeler | 12 | 7 | 22 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 46 | 21 | 10 | 68.7% (KTC 95%: 57–78%) | 82.1% (KTC 95%: 70–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 0/3
  - extra: 2/3
  - wrong_class: 2/3
  - lọt: img_029.jpg (loose), img_031.jpg (loose), img_028.jpg (extra), img_006.jpg (wrong_class), img_004.jpg (missing), img_027.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

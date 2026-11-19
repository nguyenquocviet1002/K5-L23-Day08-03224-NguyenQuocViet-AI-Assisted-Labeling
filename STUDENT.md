# Thông tin học viên

- Họ và tên: Nguyễn Quốc Việt
- MSHV: 03224
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: nguyenquocviet1002
- Repo bài nộp (URL): https://github.com/nguyenquocviet1002/K5-L23-Day08-NguyenQuocViet-03224-AI-Assisted-Labeling
- Commit chốt bài (hash): HEAD
- Máy: Windows 11 x64, Google Colab (GPU NVIDIA Tesla T4)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 571 / 8 / 43 (mẫu số: 622 box máy, tỉ lệ: 91.8% / 1.3% / 6.9%); added: 25 (mẫu số: 604 đối tượng sau duyệt, tỉ lệ: 4.1%)
- So với gold: precision / recall của **máy**: 68.7% (46/67) / 82.1% (46/56) (KTC 95%: Precision 57–78%, Recall 70–90%)
- Bắt bẫy: 6/12 = 50.0% (KTC 95%: 25–75%)
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh gold); duyệt có pre-label 1.25 phút/ảnh (30 phút / 24 ảnh batch)
- Lỗi QC: 0.0% theo ảnh (0/6 ảnh), 0.0% theo đối tượng (0/122 đối tượng)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy đạt 6/12 = 50.0% (< ngưỡng 80%) do thiên kiến tự động hóa để lọt 3 bẫy box lỏng (loose), cần rà soát lại lô và siết checklist zoom 150% kiểm biên.

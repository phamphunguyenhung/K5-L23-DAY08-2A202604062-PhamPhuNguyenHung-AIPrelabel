# Thông tin học viên

- Họ và tên: Phạm Phú Nguyễn Hưng
- MSHV: 2A202604062
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: phamphunguyenhung
- Repo bài nộp (URL): https://github.com/phamphunguyenhung/K5-L23-DAY08-2A202604062-PhamPhuNguyenHung-AIPrelabel
- Commit chốt bài (hash): 27804b7092767b9f2b04f0f4ad119e6fd99d5a29 (HEAD hiện tại, cập nhật khi chốt)
- Máy: Windows 11, 16 GB RAM, CPU (SAM chạy CPU); GPU CUDA trên Colab T4 cho YOLO26 và YOLOE
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 512 / 1 / 3 (mẫu số: 516 box máy); added: 3 (mẫu số: 516 đối tượng)
- So với gold: precision / recall của **máy** 43.8% / 58.2%
- Bắt bẫy: 10/12 = 83,3% (job 52, đo với `traps_key.json` seed 7; bẫy lọt: img_018, img_024)
- Thời gian: gán tay 2,6 phút/ảnh (10,4 phút / 4 ảnh, mốc CVAT); duyệt có pre-label 1,5 phút/ảnh (35 phút / 24 ảnh, tính cả lần 1 không đạt: tạo task 17:39, lưu cuối 18:14)
- Lỗi QC: 60,0% theo ảnh (3/5), 3,8% theo đối tượng (4/105)
- **Quyết định:** Rework — ảnh lỗi 60% (3/5) vượt ngưỡng 2%, cần sửa 4 đối tượng rồi chạy lại QC

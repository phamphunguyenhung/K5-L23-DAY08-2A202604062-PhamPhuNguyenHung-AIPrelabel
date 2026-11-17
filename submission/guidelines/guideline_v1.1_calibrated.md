# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604062 - Phạm Phú Nguyễn Hưng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện: bicycle hay motorbike? | **Chưa chốt.** Trong 5 ảnh mẫu không gặp ca này, nên không có bằng chứng để quyết. Giữ `can_review` cho đến khi gặp ca thật | — | — |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Hàng nằm trong thùng/trên xe thì thuộc box xe; hàng nhô ra quá 20% chiều dài xe thì không tính (theo mục 4.1 quy tắc 5 v1.0) | img_003 | CAL-05 |
| — | Xe đẩy tay, gánh hàng bị gọi là three_wheeler | Không gán three_wheeler; không gán lớp nào nếu không phải phương tiện | img_014, img_031 | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Một xe thồ bị vẽ 2 box three_wheeler (#11 và #15) | YOLOE/YOLO26 sinh 2 box cho cùng chiếc xe | Chỉ giữ một box; mui xe thuộc box xe | mục 4.1, quy tắc 8 (mới): một vật một box cùng lớp |
| img_014 | Người gánh hàng (#151) và xe đẩy tay (#152) bị gọi là three_wheeler | three_wheeler 0.33 và 0.63 (YOLOE) | Không gán lớp phương tiện; #151 là người gánh nên không vẽ như vật, #152 bỏ | mục 2 (lớp) và mục 4.3: không gán ba bánh cho xe đẩy tay |
| img_031 | Mũ nón treo trên kệ bị gọi là person 0.34 (#446) | person (YOLO26) | Xoá; không có người thì không vẽ | mục 4.1, quy tắc 7 (mở rộng): mũ, nón, quần áo treo không phải person |
| img_018 | Bẫy loose: box xe máy lệch khỏi vị trí gốc (IoU 0,62), chưa được sửa | motorbike (YOLO26, bẫy) | Đưa box về vị trí ôm sát xe; so với ảnh gốc | bước 0 checklist: so box với ảnh gốc sau khi sửa |
| img_024 | Bẫy missing: xe ô tô bị xoá, không vẽ lại | car (YOLO26, bẫy) | Vẽ lại box car | bước 0 checklist: quét ảnh trước khi nhìn box máy |
| img_003 | Mask SAM 2.1 xe thồ gồm trẻ ngồi, rau quả và có vệt đỏ lan lên mái và ra đường | SAM 2.1 (b), 2 click dương, 1 click âm | Mask gồm thùng, bánh, mui và hàng trong thùng; không gồm người; bỏ vệt lan | mục 4.2, quy tắc 5 (mới): người ngồi không thuộc mask xe |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 3 (hàng trên xe) và quy tắc cho xe đẩy tay, mũ/nón, box trùng, mask có người; thêm bước kiểm box sau sửa | CAL-01, CAL-02, CAL-03, CAL-04, CAL-05 |
| v1.1 | Câu 1 (xe đạp điện) chưa chốt vì chưa có ca trong mẫu | — |

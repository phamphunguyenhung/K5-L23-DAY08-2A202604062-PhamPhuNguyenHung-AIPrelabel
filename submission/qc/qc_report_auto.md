# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 10 ảnh (0 ca khó + 10 ngẫu nhiên), 192 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 3/192 = 1.6% | 192 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/10 = 20.0% | 10 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 512/516 = 99.2% / 1/516 = 0.2% / 3/516 = 0.6% | 516 box máy |
| Máy: added | 3/516 = 0.6% | 516 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_lop | 2 | img_008.jpg |
| thua | 1 | img_013.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 10 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **51%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 20.0% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.

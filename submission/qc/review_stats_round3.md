# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 516; đối tượng sau duyệt: 509; ngưỡng accepted: IoU >= 0.9
- Đã **loại 18 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 504 | 504/516 = 97.7% | 516 box máy |
| edited (sửa) | 2 | 2/516 = 0.4% | 516 box máy |
| deleted (xoá) | 10 | 10/516 = 1.9% | 516 box máy |
| added (vẽ thêm) | 3 | 3/509 = 0.6% | 509 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 3.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 271 | 0 | 1 | 0 |
| bicycle | 9 | 0 | 0 | 0 |
| motorbike | 138 | 0 | 1 | 1 |
| car | 59 | 0 | 1 | 1 |
| bus | 3 | 0 | 0 | 0 |
| truck | 7 | 0 | 0 | 0 |
| three_wheeler | 17 | 2 | 7 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 32 | 41 | 23 | 43.8% (KTC 95%: 33–55%) | 58.2% (KTC 95%: 45–70%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - extra: 3/3
  - missing: 3/3
  - wrong_class: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).

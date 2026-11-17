# FINAL AUDIT — Day 08 (2A202604062 - Phạm Phú Nguyễn Hưng)

## 1. Những việc đã thực hiện
- Snapshot job 52 trước khi sửa vòng 4 lưu ngoài repo (scratchpad: `snap/job52_before_round3.json`, `snap/job52_after_round3.json`); commit trạng thái trước khi sửa (`4211fb8`).
- Sửa annotation trong CVAT qua API (xem mục 3). Mọi thay đổi có audit trong `qc/cvat_audit_round3.csv` (before/after, bằng chứng ảnh).
- Export COCO từ CVAT ba lần: bản batch vòng 1 (`labels/batch_reviewed.json`, giữ nguyên), bản sửa QC (`labels/qc_fixed.json`), bản final mới nhất (`labels/final_round3_export.json`, 519 đối tượng).
- Chạy lại `build_release.py`, `review_stats.py` (ra `qc/review_stats_round3.*`), `qc_sample.py`, `qc_report.py`.
- Lấy mẫu QC độc lập (4 ảnh chưa từng kiểm, random seed 42), ghi `qc/qc_review_independent.csv`.
- Sửa heading `###` → `##` trong `guideline_v1.1_calibrated.md` để checker đếm được mục; nội dung không đổi.

## 2. File thay đổi (so với commit `4211fb8`)
- `submission/labels/qc_fixed.json`, `final_round3_export.json` (mới)
- `submission/release/*` (dựng lại từ CVAT)
- `submission/qc/qc_review.csv` (vòng 3), `qc_review_independent.csv` (mới), `cvat_audit_round3.csv` (mới), `review_stats_round3.*` (mới), `qc_independent_sample.json` (mới), `qc_report_auto.md`
- `submission/REPORT.md` (mục 7: vòng 3 và vòng 4), `submission/guidelines/guideline_v1.1_calibrated.md` (chỉ đổi cấp heading)
- Không sửa: `labels/gold_manual.json`, `prelabel/traps_key.json`, `scripts/check_submission.py`, `scripts/*` khác, `labels/batch_reviewed.json`.

## 3. Lỗi đã sửa và bằng chứng
| Ảnh | Shape CVAT | Thay đổi | Bằng chứng | Nguồn đánh giá |
|---|---|---|---|---|
| img_008 | 15263 (#104) | xoá | ảnh gốc: xe đẩy hàng hai bánh, không phải ba bánh | AI đọc ảnh, cần người duyệt |
| img_008 | 15264 (#105) | xoá | ảnh gốc: xe đẩy chở chuối, không động cơ | AI đọc ảnh, cần người duyệt |
| img_013 | 15289 (#128) | xoá | ảnh gốc: hai box cùng một tuk-tuk | AI đọc ảnh, cần người duyệt |
| img_013 | 15290 (#129) | kéo box về (90,28,890,682) | ảnh gốc: tuk-tuk trải qua vùng này; ước lượng | AI đọc ảnh, cần người duyệt |
| img_009 | 15687 (mới) | thêm xe máy (170,470,470,840) | ảnh gốc: xe máy không có box; loại hàng nhô ra theo 4.1.5 | AI đọc ảnh, cần người duyệt |
| img_018, img_024, img_003, img_014, img_031 | (vòng 2, trước commit `4211fb8`) | img_018 box về vị trí gốc; img_024 vẽ lại car; img_003/014/031 xoá box lỗi | xem REPORT mục 7, vòng 2 (không có file audit riêng cho vòng 2) | AI đọc ảnh + traps_key, cần người duyệt |

- **img_034 #488** không sửa: vật ở xa, bánh bị khuất; không đủ rõ để xác nhận hay bỏ. Sau đó người duyệt đã xác nhận giữ (xem `qc/qc_review.csv`).

## 4. Kết quả QC và số liệu (phân biệt các nguồn)
| Mục | Kết quả | Nguồn | Ghi chú |
|---|---|---|---|
| Vòng 1 — duyệt batch | accepted 512, edited 1, deleted 3, added 3; mẫu 516 | `qc/review_stats.json` (đã có) | lịch sử, không đổi |
| Bẫy vòng 1 | 10/12 (83,3%) | `qc/review_stats.md` | lịch sử, dùng để báo cáo |
| Bẫy trên bản final | 12/12 | `qc/review_stats_round3.md` | sau khi đã sửa img_018, img_024 — không phải năng lực duyệt ban đầu |
| Gold (máy so với gold) | TP 32, FP 41, FN 23; precision 43,8%, recall 58,2% | `review_stats*` | khớp số gốc |
| Sau QC vòng 3 (mẫu 10 ảnh, seed 42) | 3/192 đối tượng = 1,6%; 2/10 ảnh = 20% | `qc_report_auto.md` | script gợi ý REWORK |
| Mẫu độc lập (4 ảnh, 43 đối tượng) | 1 lỗi bo_sot (img_009), đã sửa; còn lại 0/43 | `qc_review_independent.csv` | AI đọc ảnh, không phải người |

**Quyết định:** Rework (giữ nguyên, chưa Release). Lý do: con số 2/10 ảnh = 20% > 2% là của mẫu vòng 3, đo trước khi sửa vòng 4; chưa có mẫu QC mới trên bản final 519 đối tượng do người kiểm; cận trên 95% của tỉ lệ ảnh lỗi ≈ 51% nên mẫu 10 ảnh chưa đủ để chứng minh ngưỡng 2%. Mẫu độc lập vòng 4 do AI soạn phiếu, người duyệt. Không Release.

## 5. Checker
- `check_submission.py`: **1 LỖI**: README phải có tiêu đề `# Day 08 Lab — <MSHV> - <Họ và tên>`. Tiêu đề hiện tại đúng định dạng đề, MSHV thật `2A202604062`. Regex trong checker dùng `\d{5,}` nên không nhận MSHV có chữ. **Đây là vấn đề của validator, không phải của bài.** Không sửa checker và không đổi MSHV theo yêu cầu đề.
- Cảnh báo: guideline không có chữ "v1.1" trong bảng lịch sử. Không sửa để tránh thay đổi nội dung bạn đã viết.

## 6. Điểm rubric (dự báo, không phải điểm chính thức)
| Mục | Tối đa | Dự báo | Độ tin cậy | Rủi ro |
|---|---|---|---|---|
| 1. Cài đặt, git | 8 | 5 | trung bình | checker còn lỗi README (validator dùng `\d`, MSHV có chữ); đã push nhánh `submission` |
| 2. Gold, label contract | 10 | 7 | cao | `can_review` không có ca thật |
| 3. Model, recipe | 20 | 14 | trung bình | chưa thử SAM 1; chưa có ngưỡng theo lớp |
| 4. Ngưỡng theo lớp | 8 | 5 | thấp | chỉ có `car=0.25`, không có ngưỡng riêng từng lớp |
| 5. Review và đo | 28 | 21 | trung bình | phân tích vòng 1 đã có; vòng 3 là AI review |
| 6. Mask SAM | 8 | 6 | cao | chưa thử Interactor CVAT |
| 7. Calibration | 10 | 8 | trung bình | quyết định do AI soạn, cần người xác nhận |
| 8. QC và báo cáo | 8 | 6 | thấp | không có QC do người thực hiện |
| **Tổng** | 100 | **~72** | | dưới 90 |

**Không thể tự tuyên bố ≥ 90.** Lý do: (a) phần phân tích và quyết định, theo rubric, cần người viết; (b) QC vòng 3 và mẫu độc lập đều do AI đọc ảnh; (c) checker còn lỗi do validator; (d) chưa có ngưỡng theo lớp.

## 7. Các mục chưa thể tự xác minh
- Mọi box sửa trong CVAT (img_008, img_013, img_009) là ước lượng từ ảnh, cần người duyệt.
- img_034 #488: người duyệt đã xác nhận giữ.
- Các vật nhỏ, bị che, hoặc ở nền tối (img_005, img_017, img_022, img_029, img_004, img_033) chưa kiểm hết từng box.
- Interactor SAM trong CVAT: chưa thử.
- Thời gian thao tác: lấy từ mốc `updated_date` của CVAT, là ước lượng, không phải bấm giờ.

## 8. Mức độ sẵn sàng nộp
- **Sẵn sàng về dữ liệu và số liệu:** có; mọi số liệu có nguồn và được đo lại từ export mới nhất.
- **Chưa sẵn sàng nộp:** checker còn 1 lỗi validator (đã chấp nhận, không sửa); QC vẫn là REWORK; chưa có QC mới do người kiểm trên bản final.
- **Đã làm:** người duyệt đã xác nhận các phiếu trong `qc_review.csv` và `qc_review_independent.csv`; nhánh `submission` đã push.

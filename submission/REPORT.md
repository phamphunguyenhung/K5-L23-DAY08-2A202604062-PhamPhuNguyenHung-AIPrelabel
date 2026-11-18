# Báo cáo Day 08 — 2A202604062 - Phạm Phú Nguyễn Hưng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào).

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | theo lớp, xem `qc/threshold_sweep.md`; lệnh merge dùng `car=0.25`, còn lại `conf 0.3` | Chạy ở ngưỡng 0.05 (r04, r06) quá nhiều box lớp ngoài taxonomy; ngưỡng 0.25–0.5 (r01, r02) gọn hơn |
| three_wheeler | YOLOE-26s-seg, prompt `three_wheeler` | ngưỡng cao hơn 0.05 (r05 chỉ dùng để tìm ứng viên) | YOLOE bắt được xe ba bánh trên 36 ảnh, nhưng gọi nhầm xe đẩy tay và người gánh hàng (r05) |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.25" --conf 0.3 --iou 0.6 --split batch,gold --out work/prelabel/merged_review/pre.coco.json` (kết quả: 598 box).
- Lưu ý `run_id`: `run.json` của `yoloe_all` ghi r03 và `y26_all` ghi r02, trùng với `work/runs.csv`. Trong `model_log.csv` tôi đánh số lại theo thứ tự chạy.
- **Phiên bản SAM đã chọn: SAM 2.1 b (`sam2.1_b.pt`).** Lý do: trọng số có sẵn trên máy, chạy được trên CPU (r03: 8,5 s/ảnh). Chưa thử SAM 1 để so, nên không kết luận SAM 2.1 tốt hơn.
- Lỗi không sửa được bằng đổi ngưỡng: xe đẩy tay (img_014 #152, 0,63) và người gánh hàng (img_014 #151, 0,33) bị gọi là three_wheeler. Điểm của chúng nằm trong khoảng điểm của xe thật, nên ngưỡng không tách được. Xử lý bằng quy tắc CAL-01.
- Box điểm cao mà sai: img_003 #15 (three_wheeler 0,90) vẽ trùng với #11 cùng một xe thồ. Box điểm thấp mà sai: img_031 #446 (person 0,34) là mũ treo trên kệ.
- Giới hạn: ngưỡng chọn trên chính gold (4 ảnh) nên số đo của máy trên gold là lạc quan.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 512 | 99,2% | 516 box máy |
| edited | 1 | 0,2% | 516 box máy |
| deleted | 3 | 0,6% | 516 box máy |
| added | 3 | 0,6% | 516 đối tượng sau duyệt |

*Nguồn: `submission/qc/review_stats.json` (bản duyệt đạt, job 52 trong CVAT, lưu cuối 18:14; 0 trường hợp đổi lớp, sót thật ≈ 3; loại 17 đối tượng quanh bẫy).*

- Lớp máy tệ nhất là three_wheeler: 24 accepted, 1 edited, 1 deleted, 1 added (mẫu số 27 box máy của lớp này). Các lỗi nhìn thấy trong QC (img_003, img_014) đều thuộc lớp này.
- So với gold: precision / recall của **máy** 43,8% / 58,2% (mẫu số: 55 đối tượng gold, 4 ảnh; TP 32, FP 41, FN 23). *Lạc quan: ngưỡng chọn trên chính gold.*
- Điều cần lưu ý: accepted 99,2% trên batch cao hơn nhiều so với precision 43,8% trên gold. Có thể tôi chấp nhận box quá nhanh khi đã có pre-label. Bằng chứng là bắt bẫy chỉ 83,3%, phần dưới.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 (83,3%, KTC 95%: 55–95%), ngưỡng đạt 80%. Theo loại: loose 2/3, extra 3/3, wrong_class 3/3, missing 2/3. Nguồn: `qc/review_stats.md` mục 3.
- Bẫy lọt (2/12):
  - img_018 (loose): box còn ở vị trí lỗi, chưa đưa về vị trí gốc (IoU với vị trí gốc 0,62).
  - img_024 (missing): xe ô tô bị xoá khi cài bẫy nhưng chưa được vẽ lại.
  - Nguyên nhân: hai lỗi này nằm ở phần box đã "trông ổn" và bỏ sót vật bị xoá. Tôi chưa so lại từng box với ảnh gốc sau khi sửa, đúng bước 0 của checklist. Điều này cho thấy khi máy đã cho sẵn nhãn, tôi dễ kiểm tra theo box thay vì theo ảnh.
- Accepted cao (99,2%) nhưng bắt bẫy thấp (83,3%): tôi chấp nhận nhãn máy nhiều hơn mức đáng có. Quy tắc ở mục 4 nhằm sửa điều này.

## 4. Máy có đáng dùng không?

- Thời gian (từ mốc CVAT, `time_log.csv`): gán tay gold 2,6 phút/ảnh (10,4 phút / 4 ảnh); duyệt có pre-label 1,5 phút/ảnh (35 phút / 24 ảnh, tính từ lúc tạo task 17:39 đến lưu cuối 18:14, gồm cả lần 1 không đạt). Thiên lệch: gold làm trước nên đã quen cảnh, nên 2,6 phút/ảnh có thể cao hơn thực tế; không tách được hiệu ứng này khỏi thời gian làm.
- Kết luận: **có, cho person / motorbike / car; chưa đủ cho three_wheeler**. Căn cứ: lớp person và motorbike có accepted cao và ít lỗi trong mẫu QC; three_wheeler có 1 deleted, 1 added và 2 lỗi sai lớp trong mẫu. Vì bắt bẫy chỉ 83,3% và precision gold 43,8%, tôi không dùng nhãn máy mà không duyệt.
- **Ba quy tắc giảm thiên kiến** (mỗi quy tắc kiểm được):
  1. Sau khi sửa box, so với ảnh gốc ít nhất một lần (CAL-04). Kiểm: 2 bẫy lọt đều thuộc loại này, nên quy tắc này cần được kiểm lại ở vòng sau.
  2. Mỗi xe một box cùng lớp (CAL-02). Kiểm: đếm box trùng IoU ≥ 0,5 cùng lớp trên mỗi ảnh; img_003 có 1 cặp trùng.
  3. Xem lại mỗi ảnh có lỗi ở lớp three_wheeler theo guideline CAL-01 trước khi duyệt. Kiểm: số lỗi sai lớp three_wheeler sau vòng sửa phải về 0 trong mẫu QC.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_035 (xích lô) | SAM 2.1 b | 2 (1 dương, 1 âm) | Gọn, bám cabin và mái; không có người trong mask | Gần như đúng | `xichlo_img035.jpg` |
| img_003 (xe thồ) | SAM 2.1 b | 12 (2 dương, 10 âm), lần 4 | Lần 1 lẫn trẻ ngồi và vệt đỏ ra đường (`sai_mask`). Lần 2 loại trẻ và vệt nhưng còn sót tay trẻ. Lần 3 thêm click âm trên tay, vẫn còn sót thân trẻ. Lần 4 thêm click âm trên thân trẻ; kiểm tra bằng pixel: thân và tay trẻ không còn phủ, thân xe, bánh và thùng vẫn phủ | Đã sửa | `xethom_img003.jpg` |
| img_031 (xe đạp chở hàng) | SAM 2.1 b | box + 3 dương + 3 âm, lần 3 | Lần 1 chỉ phủ giỏ (sai đối tượng). Lần 2 phủ cả giỏ nhãn ở giữa (hàng hoá, không thuộc xe). Lần 3 chỉ phủ khung, hai bánh và xích; loại giỏ nhãn, giỏ cam và người | Đã sửa; bánh có lỗ nhỏ (đúng quy tắc 4.2.2) | `xedap_img031.jpg` |

- Đối chiếu guideline: mục 4.2 quy tắc 1 (đường biên ôm sát) chưa đạt ở img_031 và img_003. Quy tắc còn thiếu là "người không thuộc mask xe", thêm thành quy tắc 5 trong v1.1 (CAL-05).
- Interactor SAM trong CVAT: **chưa thử**. Không so sánh được với bản offline.

## 6. Calibration (Part 6)

- Số quyết định: 5 (`qc/calibration_decisions.csv`). Quan trọng nhất: **CAL-01** (xe đẩy tay và gánh hàng không gán three_wheeler; img_014, img_031) và **CAL-02** (một xe một box; img_003).
- Quy tắc mới đã vào `guidelines/guideline_v1.1_calibrated.md`: quy tắc 8 của mục 4.1 (một vật một box cùng lớp), quy tắc 7 mở rộng (mũ, nón không phải person), quy tắc 5 của mục 4.2 (người không thuộc mask xe), và bước 0 của checklist (so box với ảnh gốc sau khi sửa).
- Câu hỏi 1 của v1.0 (xe đạp điện: bicycle hay motorbike?) **chưa chốt**: trong 5 ảnh mẫu không có ca này, nên không có bằng chứng để quyết.
- Nguyên nhân gốc: phần lớn là **máy sai** (xe đẩy tay, mũ, box trùng), và một phần là **guideline mơ hồ** (câu hỏi 3 về hàng nhô ra, mục 4.2 chưa nói về người trong mask). Bẫy lọt là lỗi của **bước kiểm** của tôi, không phải của guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 105 đối tượng.
- Người kiểm: tôi. Phiếu được soạn trước bởi AI rồi tôi đã duyệt. Đây là **hạn chế**: người vừa duyệt nhãn cũng là người kiểm mẫu; vật nhỏ (< 20 px) chưa được kiểm hết.
- Lỗi theo **đối tượng**: 4/105 = 3,8% · theo **ảnh**: 3/5 = 60,0% (ngưỡng Release ≤ 2%).
- Top lỗi: thừa 2 (img_003 #15, img_031 #446), sai lớp 2 (img_014 #151, #152). Ví dụ: img_003 cho thừa, img_014 cho sai lớp. Không có lỗi sai vị trí, sai thuộc tính hay sai mask trong mẫu này.
- **Giới hạn mẫu nhỏ:** 3/5 ảnh lỗi có cận trên 95% (Wilson) là 88%. Mẫu 5 ảnh không thể chứng minh ngưỡng 2%; chỉ đủ để thấy rằng lỗi có mặt và chưa thể nói Release.
- Script kiểm được: đủ ảnh, lớp hợp lệ, số box, tỉ lệ accepted/edited/deleted/added. Script không kiểm được: box có ôm sát vật không, lớp có đúng thực tế không, vật cả tôi lẫn máy cùng bỏ sót.

**Vòng 2 (sau khi sửa 4 đối tượng trong CVAT, export `labels/qc_fixed.json`, chạy lại build_release và qc_sample):** mẫu 5 ảnh, 101 đối tượng, lỗi 0/101 (0/5 ảnh). Đây là suy ra từ phiếu vòng 1 (các box còn lại không đổi), chưa phải lần kiểm mới độc lập. Script gợi ý Release, nhưng chưa đủ: 2 bẫy lọt (img_018, img_024) vẫn chưa sửa, và mẫu 5 ảnh có cận trên 95% là 43%.

**Vòng 3 (sau khi sửa thêm img_018 và vẽ lại img_024 trong CVAT, export `labels/qc_fixed.json` cho 5 ảnh đã sửa, mẫu tăng lên 10 ảnh, seed 42):** 192 đối tượng, lỗi 3/192 = 1,6%; 2/10 ảnh = 20,0%. Lỗi: img_008 #104 và #105 (`sai_lop`: xe đẩy hàng, không phải ba bánh); img_013 #128 (`thua`: trùng với #129 cùng một tuk-tuk). Ảnh img_034 #488 lúc đó chưa chắc; sau đó người duyệt đã xác nhận giữ (xem `qc/qc_review.csv`). Cận trên 95% của tỉ lệ ảnh lỗi là 51%, mẫu vẫn nhỏ.

**Vòng 4 (sửa CVAT lần cuối và mẫu độc lập):** xoá img_008 #104, #105 (xe đẩy hàng hai bánh) và img_013 #128 (trùng với #129); kéo #129 phủ đủ tuk-tuk. Đã kiểm tra bằng ảnh gốc và đọc lại CVAT (xem `qc/cvat_audit_round3.csv`). Mẫu độc lập gồm 4 ảnh chưa từng kiểm (img_004, img_009, img_022, img_033; 43 đối tượng, chọn bằng random seed 42): img_009 thiếu một xe máy, đã vẽ thêm; sau sửa lỗi 0/43 (AI đọc ảnh, không phải người duyệt). Phiếu mẫu 4 ảnh này nằm ở `qc/qc_review_independent.csv`. Bản final mới: 519 đối tượng (`labels/final_round3_export.json`); bản vòng 1 vẫn giữ nguyên (`labels/batch_reviewed.json`). Bẫy đo lại trên bản final: 12/12, nhưng đây là sau khi đã sửa img_018 và img_024 nên không dùng để đánh giá năng lực duyệt ban đầu (vòng 1 vẫn 10/12).

**Vòng 5 (xoá trùng):** xoá img_006 #68 (person 0.35, nằm gần trọn trong #69 cùng một người; IoU ≈ 0.43). Release sau sửa: **518 đối tượng** (`release/final_det.coco.json`, `release_summary.csv`); bản `labels/final_round3_export.json` và `labels/qc_fixed.json` cũng đã cập nhật tương ứng. So với bản 519 cũ, chỉ khác đúng box này. Lưu ý: job CVAT 52 vẫn còn box này cho đến khi xoá tay trong CVAT. Mục số 519 ở các vòng trên là số liệu lịch sử.

**Quyết định: Rework (giữ nguyên, chưa Release)**, ngày 2026-11-17. Lý do: con số 2/10 ảnh = 20% > 2% là của mẫu vòng 3, đo trước khi sửa vòng 4; chưa có mẫu QC mới trên bản final 519 đối tượng do người kiểm; cận trên 95% của tỉ lệ ảnh lỗi ≈ 51% nên mẫu 10 ảnh chưa đủ để chứng minh ngưỡng 2%. Không có vấn đề dữ liệu hỏng hay guideline chưa rõ đến mức phải Hold.

**Mẫu QC mới trên bản final (seed 2026):** 10 ảnh, 219 đối tượng (`qc/qc_sample.csv`). Phiếu người kiểm là `qc/qc_review_final_seed2026.csv`. Trạng thái: 1 ảnh (img_006) có người duyệt 4 box nghi ngờ, 9 ảnh còn lại **Pending Human Review**. Đây là số liệu duy nhất, không có tỉ lệ lỗi mới để tính ngưỡng. Ứng viên cần xem lại (ảnh phóng to có ID): `work/qc/candidates/` (5 ảnh). Lưu ý: ID trong `work/qc/review/` vẽ trước khi xoá #68 nên lệch 1 sau img_006; dùng ID trong `release/final_det.coco.json` hiện tại. Kiểm AI sơ bộ 5 ảnh (img_006, img_013, img_018, img_022, img_023) trong `qc/qc_ai_spotcheck_final.csv`: không thấy lỗi lớp rõ ràng, nhưng đây là AI xem ảnh, không phải người, và chưa kiểm từng box.

Việc còn lại: (1) người kiểm điền phiếu mới cho mẫu seed 2026 vào một file riêng, ví dụ `qc/qc_review_final_seed2026.csv` (không ghi đè `qc_review.csv`, vì file đó vẫn là phiếu vòng 3–4); (2) chạy `qc_report.py` để tính tỉ lệ rework mới; (3) nếu tỉ lệ ảnh lỗi ≤ 2% thì mới xét Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude (Claude Code) soạn nháp phiếu QC, calibration, guideline v1.1, `model_log.csv` và báo cáo này từ số liệu trong repo và các file CVAT. Tôi đã đọc và duyệt từng phần. Mask SAM chạy bằng `scripts/sam2_points.py` trên CPU, không chạy Colab. Số liệu được kiểm bằng `review_stats.py`, `qc_report.py` và `check_submission.py`.
- Gold có bị nhiễm không: không. Gold gán trước (file `gold_manual.json` lưu lúc 17:24), còn pre-label và pre-traps được tạo sau mốc đó.
- Máy / GPU / CVAT: Windows 11, 16 GB RAM, CPU cho SAM; GPU CUDA cho YOLO26 và YOLOE (chạy trên Colab T4 theo `run.json`); CVAT 2.74.1 chạy Docker local. Dùng cache `data/cache/`: không, model chạy trực tiếp.
- Lỗi / bug script đã gặp: API CVAT cần header `Authorization: Bearer <token>`, không phải `Token`; đây là điều tôi đã sai khi thử đọc job lúc đầu. Export COCO từ CVAT phải lấy `result_url` của request rồi mới tải được file.
- Một điều làm khác lần sau: ghi mốc thời gian (mở, lưu cuối) của từng job ngay khi làm, không đo lại sau; kiểm từng box với ảnh gốc sau khi sửa trước khi chuyển sang box tiếp theo.

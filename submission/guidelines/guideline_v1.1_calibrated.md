# LABEL CONTRACT – PHỐ CHÂU Á
## BỔ SUNG PHIÊN BẢN 1.1

**Họ và tên:** Phạm Phú Nguyễn Hưng  
**Mã học viên:** 2A202604062

### 1. Các nội dung được cập nhật từ phiên bản 1.0

Sau khi thực hiện gán nhãn và kiểm tra chất lượng dữ liệu, tôi đã rà soát lại một số trường hợp chưa được quy định rõ trong phiên bản 1.0. Dựa trên các ảnh thực tế và kết quả kiểm tra, tôi bổ sung các quyết định sau:

**1.1. Phân loại xe đạp điện**

Hiện tại tôi chưa đưa ra quyết định cuối cùng về việc xếp xe đạp điện vào lớp `bicycle` hay `motorbike`, vì trong 5 ảnh được kiểm tra chưa xuất hiện trường hợp này.

Tạm thời, những đối tượng tương tự sẽ được đánh dấu `can_review` để tiếp tục xem xét khi có ảnh thực tế, tránh tự ý phân loại khi chưa có đủ căn cứ.

**1.2. Xác định bounding box đối với xe chở hàng**

Trong ảnh `img_003` (CAL-05), tôi nhận thấy cần xác định rõ phạm vi bounding box của xe chở hàng.

Tôi thống nhất giữ phần hàng nằm trong thùng hoặc trên xe trong phạm vi bounding box. Đối với hàng hóa nhô ra quá 20% chiều dài xe, phần nhô ra sẽ không được tính vào box theo quy tắc đã nêu tại mục 4.1 của phiên bản 1.0.

**1.3. Phân biệt xe ba bánh và phương tiện vận chuyển thủ công**

Trong các ảnh `img_014` và `img_031` (CAL-01), mô hình nhận diện nhầm người gánh hàng hoặc xe đẩy tay thành `three_wheeler`.

Tôi bổ sung quy định không sử dụng nhãn `three_wheeler` cho các phương tiện đẩy tay hoặc vật dụng vận chuyển không thuộc định nghĩa của lớp này. Nếu đối tượng không phù hợp với bất kỳ lớp nào trong bộ nhãn thì không tạo bounding box.

### 2. Các quy tắc bổ sung sau quá trình kiểm tra

Qua kiểm tra kết quả gán nhãn tự động, tôi ghi nhận một số lỗi của mô hình và những trường hợp bản thân còn bỏ sót. Từ đó, tôi bổ sung các quy tắc sau để việc gán nhãn được nhất quán hơn.

**2.1. Không tạo nhiều bounding box cho cùng một đối tượng**

- Ảnh minh họa: `img_003`.
- Vấn đề: YOLOE/YOLO26 tạo hai bounding box (#11 và #15) cho cùng một xe thồ.
- Cách xử lý: Chỉ giữ lại một bounding box bao phủ đúng chiếc xe, bao gồm cả phần mui xe.
- Quy tắc bổ sung: Mỗi đối tượng chỉ được có một bounding box thuộc cùng một lớp, tránh tạo nhãn trùng lặp.

Quy tắc này được bổ sung vào mục 4.1, quy tắc 8.

**2.2. Không nhầm xe đẩy tay và người gánh hàng với xe ba bánh**

- Ảnh minh họa: `img_014`.
- Vấn đề: YOLOE nhận diện người gánh hàng (#151) và xe đẩy tay (#152) thành `three_wheeler`, với độ tin cậy lần lượt là 0,33 và 0,63.
- Cách xử lý: Loại bỏ các bounding box không đúng lớp phương tiện. Không tạo box phương tiện cho người gánh hàng hoặc xe đẩy tay.
- Quy tắc bổ sung: Chỉ sử dụng nhãn `three_wheeler` cho những đối tượng thực sự đáp ứng định nghĩa của lớp xe ba bánh.

Nội dung này được bổ sung vào mục 2 và mục 4.3.

**2.3. Phân biệt người thật với đồ vật có hình dạng tương tự**

- Ảnh minh họa: `img_031`.
- Vấn đề: YOLO26 nhận diện nhầm mũ, nón treo trên kệ thành `person` (#446), với độ tin cậy 0,34.
- Cách xử lý: Xóa bounding box vì trong vùng đó không có người.
- Quy tắc bổ sung: Không gán nhãn `person` cho mũ, nón, quần áo hoặc các vật dụng treo có hình dạng dễ gây nhầm lẫn với con người.

Quy tắc này được bổ sung vào mục 4.1, mở rộng quy tắc 7.

**2.4. Kiểm tra lại độ chính xác của bounding box sau khi chỉnh sửa**

- Ảnh minh họa: `img_018`.
- Vấn đề: Một bounding box xe máy bị dịch lệch so với vị trí thực tế, có IoU bằng 0,62 nhưng chưa được phát hiện và sửa lại.
- Cách xử lý: Điều chỉnh bounding box để ôm sát chiếc xe và đối chiếu lại với ảnh gốc.
- Bài học rút ra: Không chỉ kiểm tra tên lớp mà còn phải quan sát vị trí và kích thước bounding box.

Tôi bổ sung bước đối chiếu bounding box với ảnh gốc vào checklist QC.

**2.5. Kiểm tra các đối tượng bị bỏ sót**

- Ảnh minh họa: `img_024`.
- Vấn đề: Một bounding box thuộc lớp `car` bị xóa trong bộ kiểm thử nhưng tôi chưa phát hiện để vẽ lại.
- Cách xử lý: Bổ sung lại bounding box cho chiếc ô tô bị thiếu.
- Bài học rút ra: Nếu chỉ tập trung vào những bounding box đã có sẵn, người gán nhãn dễ bỏ qua các đối tượng chưa được đánh dấu.

Tôi bổ sung bước quan sát toàn bộ ảnh để tìm đối tượng trước khi kiểm tra các box do mô hình tạo ra.

**2.6. Không đưa người ngồi trên xe vào segmentation mask của phương tiện**

- Ảnh minh họa: `img_003`.
- Vấn đề: Khi sử dụng SAM 2.1 với hai điểm chọn dương và một điểm chọn âm, mask của xe thồ bao gồm cả trẻ em ngồi trên xe, rau quả và một số vùng bị lan ra mái, mặt đường.
- Cách xử lý: Chỉnh sửa mask để chỉ giữ lại thân xe, bánh xe, mui xe và hàng hóa nằm trong thùng. Loại bỏ người ngồi trên xe và các vùng mask bị lan ra ngoài.
- Quy tắc bổ sung: Khi phân đoạn phương tiện, người ngồi trên xe phải được tách khỏi mask của phương tiện. Đồng thời cần kiểm tra lại đường biên để tránh mask bao phủ sang những vùng không liên quan.

Quy tắc này được bổ sung vào mục 4.2, quy tắc 5.

### 3. Tổng kết thay đổi

So với phiên bản 1.0, tôi đã bổ sung và làm rõ các nội dung sau:

1. Quy định phạm vi bounding box đối với xe chở hàng và hàng hóa nhô ra ngoài.
2. Phân biệt xe ba bánh với xe đẩy tay và các hình thức vận chuyển thủ công.
3. Không tạo bounding box trùng nhau cho cùng một đối tượng.
4. Tránh nhận diện nhầm mũ, nón và quần áo thành người.
5. Kiểm tra vị trí bounding box và các đối tượng bị bỏ sót trong quá trình QC.
6. Tách người khỏi mask phương tiện và xử lý các vùng phân đoạn bị lan.

Các thay đổi được tổng hợp từ những trường hợp thực tế trong các ảnh `img_003`, `img_014`, `img_018`, `img_024` và `img_031`, đồng thời đối chiếu với các quyết định đã ghi nhận trong quá trình calibration.

Riêng câu hỏi phân loại xe đạp điện vẫn chưa được chốt do chưa có ví dụ phù hợp trong mẫu kiểm tra. Tôi sẽ tiếp tục sử dụng trạng thái `can_review` cho trường hợp này.

### 4. Kết luận

Qua quá trình thực hiện, tôi nhận thấy mô hình tự động giúp giảm đáng kể công việc gán nhãn ban đầu, nhưng vẫn có thể nhận diện nhầm đối tượng, tạo box trùng, bỏ sót phương tiện hoặc phân đoạn sai ranh giới.

Ngoài lỗi từ mô hình, quá trình kiểm tra của người gán nhãn cũng có thể bỏ sót lỗi nếu chỉ tập trung chỉnh sửa các box có sẵn. Vì vậy, tôi bổ sung bước quan sát ảnh gốc trước khi kiểm tra nhãn và đối chiếu lại kết quả sau khi chỉnh sửa.

Phiên bản 1.1 tập trung khắc phục những vấn đề đã gặp và làm rõ các quy tắc còn thiếu, nhằm giúp quá trình gán nhãn và kiểm tra chất lượng ở những lần tiếp theo chính xác, thống nhất hơn.
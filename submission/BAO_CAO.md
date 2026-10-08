# BÁO CÁO LAB: SO SÁNH MULTI-OBJECT TRACKING TRÊN 5 VIDEO

**Nhóm:** Không biết tên gì
**Thành viên:** Bùi Đức Thành — 2A202602364

## 1. Cấu hình đã chọn

### 1.1. Môi trường và phương pháp

Bài thực hành sử dụng detector YOLO26n (`yolo26n.pt`) với kích thước đầu vào 640 px, chỉ phát hiện lớp `person`. Các tracker được thử nghiệm gồm ByteTrack và BoT-SORT. Với tracker sử dụng đặc trưng ngoại hình, trọng số Re-ID được cố định theo cấu hình bài lab là `osnet_x0_25_msmt17.pt`.

Các thí nghiệm được thực hiện bằng cách thay đổi tracker, ngưỡng confidence và ngưỡng IoU NMS của detector. Mỗi lượt thử nhanh xử lý 150 frame, sau đó lựa chọn cấu hình phù hợp và chạy lại toàn bộ frame để tạo file nộp.

Việc lựa chọn dựa trên quan sát mức độ ổn định của bounding box, khả năng duy trì ID, hiện tượng ID switch, mất dấu đối tượng và phát hiện sai.

### 1.2. Bảng cấu hình cuối cùng

| Video | Tracker | Conf | IoU | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 — Quảng trường ban ngày | BoT-SORT | 0.15 | 0.5 | Giảm tình trạng box nhấp nháy, phần lớn đối tượng giữ nguyên ID sau khi xuất hiện lại | ByteTrack conf=0.3; BoT-SORT conf=0.3/0.5; IoU=0.7 |
| video_2 — Phố đêm đông người | BoT-SORT | 0.15 | 0.5 | Giữ ID tốt hơn ByteTrack ở cùng ngưỡng, giảm hiện tượng mất box liên tục | ByteTrack conf=0.3/0.15; BoT-SORT conf=0.3 |
| video_3 — Camera di chuyển, FPS thấp | BoT-SORT | 0.15 | 0.5 | Giữ ID ổn định, phát hiện thêm người khi giảm confidence | ByteTrack conf=0.3; BoT-SORT conf=0.3; IoU=0.4/0.7 |
| video_4 — Trong nhà, phản chiếu kính | BoT-SORT | 0.15 | 0.5 | Giữ ID tốt hơn ByteTrack, ít mất box, chưa quan sát thấy phát hiện nhầm hình phản chiếu | ByteTrack conf=0.3; BoT-SORT conf=0.3/0.5; IoU=0.4/0.7 |
| video_5 — Camera trên xe bus | ByteTrack | 0.15 | 0.4 | Tracking cải thiện một phần sau khi tinh chỉnh nhưng vẫn còn ID switch | BoT-SORT conf=0.3; các ngưỡng conf/IoU khác của ByteTrack |

### 1.3. Kết quả chạy toàn bộ dữ liệu

| Video | Số frame | Số dòng tracking | File kết quả |
|---|---:|---:|---|
| video_1 | 600 | 4,804 | `video_1.txt` |
| video_2 | 1,050 | 13,692 | `video_2.txt` |
| video_3 | 837 | 4,891 | `video_3.txt` |
| video_4 | 900 | 6,409 | `video_4.txt` |
| video_5 | 750 | 2,183 | `video_5.txt` |

Cả năm file kết quả đều tồn tại trong thư mục `runs/nop_bai/` và đều có bản ghi tracking tại frame cuối của từng chuỗi. Số dòng tracking không tương ứng trực tiếp với số frame vì một frame có thể chứa nhiều đối tượng.

## 2. Số liệu đánh giá video_1

Video 1 là chuỗi duy nhất có ground truth trong gói dữ liệu học viên, cho phép đánh giá định lượng bằng TrackEval.

**Cấu hình được đánh giá:**

- Tracker: BoT-SORT
- Confidence: 0.15
- IoU NMS: 0.5
- Số frame: 600
- File dự đoán: `runs/nop_bai/video_1.txt`
- Ground truth: `video_1/gt/gt.txt`

### 2.1. Kết quả HOTA, MOTA, IDF1

| Metric | Kết quả |
|---|---:|
| **HOTA** | **29.167%** |
| **MOTA** | **20.677%** |
| **IDF1** | **30.230%** |
| Detection Accuracy (DetA) | 18.636% |
| Association Accuracy (AssA) | 45.899% |
| Detection Recall | 22.561% |
| Detection Precision | 92.846% |
| Localization Accuracy (LocA) | 83.741% |
| MOTP | 81.238% |
| ID Switches (IDSW) | 27 |
| Fragmentations (Frag) | 65 |

### 2.2. Thống kê detection

| Chỉ số | Giá trị |
|---|---:|
| Ground-truth detections | 18,581 |
| Tracker detections sau tiền xử lý | 4,515 |
| True Positives (TP) | 4,192 |
| False Negatives (FN) | 14,389 |
| False Positives (FP) | 323 |
| Ground-truth IDs | 62 |
| Tracker IDs | 54 |

### 2.3. Phân tích kết quả

**HOTA = 29.167%:** Chất lượng tổng thể của hệ thống tracking còn hạn chế. Chỉ số Detection Accuracy (18.636%) thấp hơn Association Accuracy (45.899%), cho thấy việc phát hiện đủ đối tượng là một vấn đề đáng kể của pipeline.

**MOTA = 20.677%:** MOTA chịu ảnh hưởng lớn từ số lượng False Negatives. Với 14,389 FN, hệ thống bỏ sót nhiều đối tượng được đánh dấu trong ground truth. Đây là một trong những nguyên nhân chính khiến MOTA thấp, bên cạnh 323 FP và 27 ID switches.

**IDF1 = 30.230%:** Khả năng duy trì đúng danh tính vẫn chưa tốt trên toàn bộ chuỗi video, dù khi quan sát một số đoạn ngắn, nhiều đối tượng có thể giữ ID tương đối ổn định.

**Detection Precision = 92.846%:** Những bounding box được tracker đưa ra có tỷ lệ ghép đúng khá cao theo đánh giá CLEAR MOT.

**Detection Recall = 22.561%:** Hệ thống chỉ phát hiện và ghép đúng được một phần nhỏ số đối tượng trong ground truth. Một nguyên nhân có thể là người ở xa có kích thước nhỏ, gây khó khăn cho detector YOLO26n với đầu vào 640 px.

**ID Switches = 27:** Vẫn xảy ra các trường hợp gán ID không nhất quán khi theo dõi đối tượng qua nhiều frame.

Nhìn chung, kết quả cho thấy hệ thống có precision cao nhưng recall thấp. Để cải thiện chất lượng tracking, không chỉ cần tối ưu thuật toán liên kết ID mà còn phải cải thiện khả năng phát hiện người trong các tình huống khó.

### 2.4. Ghi chú về phương pháp đánh giá

Gói dữ liệu cung cấp `gt.txt` và `seqinfo.ini` nhưng không chứa `eval_config.json` mà script `evaluate_practice.py` yêu cầu.

Để hoàn thành bước đánh giá cục bộ, nhóm tạo cấu hình MOT17-style với `benchmark=MOT17`, `split=train` và sử dụng TrackEval với tiền xử lý lớp pedestrian.

Kết quả trên được tính từ dữ liệu ground truth thực tế và file tracking của nhóm, **không phải số liệu giả lập**. Tuy nhiên, do cấu hình benchmark được tự thiết lập, nhóm chưa xác nhận kết quả này hoàn toàn tương đương với cấu hình chấm điểm chính thức của giảng viên.

## 3. Phân tích từng video

### 3.1. Video 1 — Quảng trường ban ngày, camera tĩnh

Video 1 có camera cố định và mật độ người vừa phải, tạo điều kiện thuận lợi hơn cho các thuật toán theo dõi dựa trên chuyển động. Qua các lượt thử, BoT-SORT cho kết quả quan sát tốt khi giảm confidence từ 0.3 xuống 0.15. Số lần bounding box biến mất giảm và phần lớn đối tượng giữ được ID khi xuất hiện lại sau một khoảng mất dấu ngắn.

Khi tăng confidence lên 0.5, số đối tượng bị bỏ sót tăng. Khi tăng IoU NMS lên 0.7, xuất hiện nhiều box trùng hoặc nhấp nháy hơn. Vì vậy, nhóm lựa chọn BoT-SORT với confidence 0.15 và IoU 0.5.

Kết quả TrackEval cho thấy chất lượng thực tế còn hạn chế, đặc biệt ở Detection Recall. Điều này nhấn mạnh sự khác biệt giữa đánh giá trực quan trên đoạn video ngắn và đánh giá định lượng trên toàn bộ ground truth.

### 3.2. Video 2 — Phố đêm, mật độ rất đông

Video 2 có camera cố định trên cao, điều kiện ánh sáng yếu và nhiều đối tượng di chuyển sát nhau. Những đặc điểm này làm tăng nguy cơ detection bị gián đoạn và ID bị thay đổi khi hai người đi cắt ngang nhau.

Ở confidence 0.3, cả ByteTrack và BoT-SORT đều có hiện tượng nhấp nháy. BoT-SORT phát hiện và theo dõi nhiều đối tượng hơn nhưng chưa ổn định. Khi giảm confidence xuống 0.15, BoT-SORT giữ ID tốt hơn và giảm mất box rõ rệt. Khi so sánh ByteTrack và BoT-SORT ở cùng confidence 0.15, BoT-SORT vẫn cho kết quả quan sát tốt hơn.

Nhóm lựa chọn BoT-SORT với confidence 0.15 và IoU 0.5. Đặc trưng ngoại hình Re-ID có thể hỗ trợ liên kết danh tính trong tình huống che khuất, dù video này không có ground truth để xác nhận bằng metric.

### 3.3. Video 3 — Camera di chuyển, ảnh nhỏ, FPS thấp

Video 3 có chuyển động camera và tốc độ lấy mẫu frame thấp. Vì vậy, vị trí của một đối tượng có thể thay đổi đáng kể giữa hai frame liên tiếp, làm cho việc dự đoán chuyển động trở nên khó khăn.

Cả ByteTrack và BoT-SORT đều cho kết quả tương đối ổn định trong đoạn thử 150 frame, nhưng BoT-SORT giữ ID nhỉnh hơn theo quan sát. Khi giảm confidence từ 0.3 xuống 0.15, hệ thống phát hiện thêm người mà vẫn duy trì tracking ổn định.

Sau khi so sánh các ngưỡng IoU NMS 0.4, 0.5 và 0.7, nhóm lựa chọn 0.5. Cấu hình cuối cùng là BoT-SORT với confidence 0.15 và IoU 0.5.

### 3.4. Video 4 — Trong nhà, camera tiến tới và có phản chiếu kính

Video 4 có camera di chuyển tiến về phía trước, khiến vị trí và kích thước bounding box của các đối tượng thay đổi liên tục. Ngoài ra, các bề mặt kính có nguy cơ tạo ra hình phản chiếu khiến detector phát hiện nhầm.

Qua các lượt thử, BoT-SORT giữ ID tốt hơn ByteTrack nhưng vẫn xuất hiện mất box ở confidence 0.3. Khi giảm confidence xuống 0.15, tracking được cải thiện và ít xảy ra mất box hơn.

Nhóm không quan sát thấy trường hợp rõ ràng nhận nhầm hình phản chiếu trong các đoạn kiểm tra. Sau khi thử các ngưỡng IoU 0.4, 0.5 và 0.7, nhóm tiếp tục lựa chọn IoU 0.5.

Cấu hình cuối cùng là BoT-SORT, confidence 0.15 và IoU 0.5.

### 3.5. Video 5 — Góc nhìn từ xe bus, camera rung lắc

Video 5 là cảnh có chuyển động camera đáng kể do góc nhìn từ xe bus tại giao lộ. Các đối tượng và bối cảnh đều thay đổi vị trí nhanh giữa các frame, gây khó khăn cho việc dự đoán chuyển động và liên kết danh tính.

Khác với bốn video trước, ByteTrack giữ ID tốt hơn BoT-SORT trong lượt so sánh ở cùng confidence 0.3 và IoU 0.5. Tuy nhiên, cả hai tracker vẫn gặp hiện tượng bounding box nhấp nháy.

Sau khi thử các mức confidence và IoU khác nhau, nhóm chọn ByteTrack với confidence 0.15 và IoU 0.4. Cấu hình này cải thiện một phần kết quả tracking, nhưng vẫn có ID switches.

Kết quả cho thấy tracker sử dụng Re-ID không phải lúc nào cũng vượt trội so với tracker thiên về chuyển động. Hiệu quả phụ thuộc vào đặc điểm cảnh, chất lượng detection và mức độ ổn định của hình ảnh.

## 4. Nếu có thêm thời gian

Nếu có thêm thời gian, nhóm sẽ thử OC-SORT, StrongSORT và DeepOCSORT để so sánh khả năng liên kết danh tính trong điều kiện camera chuyển động mạnh hoặc đối tượng bị che khuất.

Ngoài ra, nhóm muốn kiểm tra kỹ các frame xảy ra ID switch, đánh giá tác động của ngưỡng confidence và phân tích những trường hợp False Negatives của video 1.

Một hướng cải thiện khác là thử detector mạnh hơn hoặc thay đổi độ phân giải đầu vào trong thí nghiệm mở rộng. Tuy nhiên, các thay đổi này không thuộc cấu hình được phép của bài nộp chính, vì detector và kích thước ảnh đã được giảng viên cố định.

## 5. Kết luận

Bài thực hành đã thử nghiệm và lựa chọn cấu hình tracking cho năm video với các điều kiện khác nhau, bao gồm camera tĩnh, cảnh đông người, thiếu sáng, chuyển động camera và phản chiếu kính.

BoT-SORT được lựa chọn cho bốn video đầu tiên, trong khi ByteTrack được lựa chọn cho video 5. Confidence 0.15 thường cải thiện độ liên tục của detection trong các lượt thử, nhưng hiệu quả cần được đánh giá riêng theo từng cảnh.

Đối với video 1, kết quả TrackEval cục bộ đạt HOTA 29.167%, MOTA 20.677% và IDF1 30.230%. Kết quả chỉ ra rằng khả năng phát hiện đầy đủ đối tượng còn là hạn chế lớn của pipeline.

Qua bài thực hành, nhóm nhận thấy **không có một tracker hoặc bộ ngưỡng nào luôn tối ưu cho mọi tình huống**. Việc lựa chọn cần dựa trên đặc điểm chuyển động, mật độ đối tượng, ánh sáng và quan sát thực nghiệm, đồng thời kết hợp metric định lượng khi có ground truth.

**Ghi chú môi trường:** Nhóm sử dụng Python 3.11, CUDA trên RTX 3050 và BoxMOT 10.0.50 để khắc phục xung đột cài đặt trên Windows. Phiên bản BoxMOT này khác phiên bản 10.0.42 được khai báo trong requirements ban đầu.

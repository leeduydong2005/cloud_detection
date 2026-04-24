# CLOUD_DETECTION_USE_ML
## 1. Giới thiệu (Introduction)

Phát hiện và phân loại mây trên ảnh vệ tinh là một bước tiền xử lý thiết yếu trong lĩnh vực viễn thám. Việc loại bỏ chính xác các vùng bị mây che phủ giúp nâng cao độ tin cậy của các phân tích thứ cấp như đánh giá thảm thực vật, ước lượng nhiệt độ bề mặt và giám sát biến đổi khí hậu.

Dự án này đề xuất một hệ thống phân loại mây ở cấp độ điểm ảnh (pixel-level) dựa trên dữ liệu quang học đa phổ của vệ tinh Sentinel-2. Điểm đột phá của nghiên cứu nằm ở việc tiếp cận bài toán xử lý ảnh thông qua các kỹ thuật Học máy trên Dữ liệu Bảng (Tabular Machine Learning), kết hợp với quy trình trích xuất đặc trưng vật lý chuyên sâu và tối ưu hóa ngưỡng phân loại.

## 2. Dữ liệu Nghiên Cứu (Dataset)

Nguồn dữ liệu được thu thập từ nền tảng Google Earth Engine (GEE), tập trung vào khu vực Hà Nội trong năm 2023. Để đảm bảo tính đại diện và khả năng tổng quát hóa, dữ liệu được thu thập ở cả hai mùa (mùa Hạ và mùa Đông), đối phó với sự đa dạng về điều kiện chiếu xạ và mức độ che phủ mây.

* **Tập huấn luyện (Training Set):** Hơn 25.4 triệu điểm ảnh, được cân bằng chặt chẽ giữa hai lớp (mây và không mây).
* **Tập kiểm thử (Test Set):** Hơn 12 triệu điểm ảnh, được trích xuất từ hai ngày chụp độc lập (28/05 và 09/12) và được gán nhãn thủ công (Ground Truth) để đảm bảo độ chính xác tuyệt đối trong khâu đánh giá.

## 3. Phương Pháp Luận (Methodology)

### 3.1. Trích Xuất Đặc Trưng (Feature Engineering)
Mỗi điểm ảnh được biểu diễn bởi một không gian đặc trưng gồm 22 chiều:
* **12 dải phổ gốc** từ Sentinel-2 (B01 đến B12).
* **10 chỉ số quang học phái sinh** (như NDVI, NDWI, NDSI) và các tỷ số phổ (như B02/B04). Các chỉ số này được thiết kế để định lượng các tính chất vật lý của bề mặt (độ ẩm, sinh khối thực vật) và tận dụng sự khác biệt về khả năng phản xạ, hấp thụ bức xạ giữa mây và bề mặt Trái Đất.

### 3.2. Kiến Trúc Huấn Luyện (Training Architecture)
Để đối phó với hiện tượng "rò rỉ không gian" (spatial data leakage) đặc trưng trong dữ liệu viễn thám, dự án áp dụng chiến lược **Group K-Fold Cross-Validation**. Dữ liệu được chia theo nhóm ở cấp độ khung ảnh (tile) hoặc ảnh vệ tinh hoàn chỉnh, buộc mô hình phải học các quy luật vật lý tổng quát thay vì ghi nhớ đặc trưng không gian cục bộ.

Các mô hình được triển khai và so sánh bao gồm: Random Forest, XGBoost, LightGBM, CatBoost và Linear SVM. Trong đó, mô hình XGBoost cho thấy hiệu năng vượt trội nhờ khả năng nắm bắt tốt các tương quan phi tuyến giữa các đặc trưng quang học.

### 3.3. Tối Ưu Hóa Ngưỡng (Threshold Optimization)
Mô hình xuất ra xác suất (hat{p}) điểm ảnh là mây. Thông qua việc đánh giá trên tập dự đoán ngoài mẫu (Out-Of-Fold), hệ thống không sử dụng ngưỡng mặc định 0.5 mà áp dụng ngưỡng tối ưu là 0.35. Việc hạ ngưỡng quyết định này giúp tối đa hóa chỉ số F_2-score, ưu tiên việc thu hồi (Recall) các điểm ảnh mây mỏng và vùng biên mây, nhằm hạn chế rủi ro bỏ sót mây trong các ứng dụng thực tiễn.

## 4. Kết Quả (Results)

Hệ thống đã chứng minh được hiệu năng cao và tính ổn định trên tập kiểm thử độc lập:
* **Hiệu năng tổng thể:** Mô hình XGBoost đạt F_1-score 0.8878.
* **Tính ổn định theo mùa:** Mô hình duy trì khả năng nhận diện tốt ở cả mùa Hạ (dữ liệu có độ tương phản cao) và mùa Đông (điều kiện chiếu xạ yếu, tỷ lệ mây cao).
* **Mô hình kết hợp (Voting Ensemble):** Mặc dù điểm số tuyệt đối không vượt XGBoost, phương pháp kết hợp các dự đoán (Hard Voting) giúp giảm phương sai, gia tăng độ ổn định của hệ thống khi triển khai trên các khu vực địa lý mới.

## 5. Cấu Trúc Mã Nguồn (Repository Structure)

Quá trình triển khai được chia thành các module tuần tự, đảm bảo tính tái lập (reproducibility) của nghiên cứu:

* `S2Cloud.ipynb`: Khảo sát ảnh thô và tính toán thống kê dải phổ.
* `create_data.ipynb`: Xử lý không gian, phân rã ảnh thành các Tile, lọc và trích xuất dữ liệu dạng bảng.
* `visualize.ipynb`: Khám phá dữ liệu (EDA), phân tích ma trận tương quan giữa các đặc trưng và nhãn lớp.
* `update_data.ipynb`: Cung cấp các tiện ích xử lý, làm sạch và hiệu chỉnh nhãn dữ liệu.
* `train.ipynb`: Triển khai huấn luyện các mô hình Machine Learning theo chiến lược Group K-Fold.
## 6. Hướng Dẫn Cài Đặt và Sử Dụng (Installation and Usage)

Hệ thống được thiết kế để vận hành trên nền tảng **Google Colaboratory (Colab)** nhằm tận dụng tài nguyên GPU miễn phí và kết nối trực tiếp với **Google Drive** để xử lý các tệp ảnh vệ tinh dung lượng lớn. 

Dưới đây là luồng thao tác (pipeline) chi tiết dành cho người mới sử dụng:

### 6.1. Chuẩn Bị Môi Trường (Setup)
1. **Lưu trữ mã nguồn:** Tải toàn bộ các file `.ipynb` của dự án này và upload vào một thư mục trên Google Drive của bạn (ví dụ: `MyDrive/Cloud_Detection`).
2. **Chuẩn bị dữ liệu:** Tải các ảnh vệ tinh Sentinel-2 gốc (định dạng `.tif`) được thu thập từ Google Earth Engine vào một thư mục con (ví dụ: `MyDrive/Cloud_Detection/Data`).
3. **Mở Google Colab:** Nhấp đúp vào tệp `.ipynb` bất kỳ và chọn *Mở bằng Google Colaboratory*. 
4. **Bật GPU:** Trên thanh công cụ của Colab, chọn `Runtime` (Thời gian chạy) > `Change runtime type` (Thay đổi loại thời gian chạy) > Chọn `T4 GPU` > Bấm `Save`.

### 6.2. Cấu Hình Chung (Dành cho mọi file)
Trong tất cả các tệp Notebook, việc đầu tiên bạn luôn phải làm là:
1. Chạy đoạn code để kết nối Colab với Google Drive:
   ```python
   from google.colab import drive
   drive.mount('/content/drive')
   ```

2. Cài đặt các thư viện bổ sung (nếu môi trường Colab báo thiếu module):
   ```python
   !pip install rasterio pytorch-tabnet catboost lightgbm xgboost
   ```

### 6.3. Trình Tự Thực Thi (Execution Pipeline)

Để cho ra kết quả dự đoán cuối cùng, bạn cần mở và chạy lần lượt các tệp theo thứ tự sau:

#### Bước 1: Khởi tạo tập dữ liệu huấn luyện (`create_data.ipynb`)
* **Mục đích:** Băm nhỏ ảnh `.tif` gốc và trích xuất đặc trưng thành dữ liệu bảng.
* **Thao tác:** * Tìm biến `FILE_TO_PROCESS` và sửa thành đường dẫn trỏ tới file ảnh `.tif` của bạn.
  * Tìm biến `OUTPUT_DIR` và sửa thành đường dẫn thư mục mà bạn muốn lưu file dữ liệu.
* **Kết quả:** Hệ thống sẽ chạy và xuất ra một tệp `[Tên_Ảnh]_data.csv`.

#### Bước 2: Phân tích và làm sạch dữ liệu (Tùy chọn)
* Mở tệp `visualize.ipynb`, nạp file `.csv` vừa tạo vào để xem các biểu đồ phân tích mức độ tương quan giữa các băng tần quang phổ với mây.
* Mở tệp `update_data.ipynb` nếu bạn cần xóa bớt các tọa độ nhiễu hoặc gán lại nhãn (relabel) cho tệp CSV để dữ liệu sạch hơn.

#### Bước 3: Tinh chỉnh và Huấn luyện mô hình (`turning.ipynb`)
* **Mục đích:** Huấn luyện thử nghiệm các thuật toán để tìm ra cấu hình mạnh nhất.
* **Thao tác:** * Chỉnh sửa đường dẫn đọc file `.csv` (ở đầu file).
  * Chỉnh sửa biến `OUTPUT_PATH` (hoặc `MODEL_DIR`) trỏ về thư mục mà bạn muốn lưu mô hình.
* **Kết quả:** Code sẽ chạy và tự động lưu lại các tệp trọng số (ví dụ: `.pkl`, `.json`, `.cbm`) của các thuật toán có hiệu năng cao nhất vào thư mục Models.

#### Bước 4: Dự đoán mặt nạ mây thực tế (`train.ipynb`)
* **Mục đích:** Dùng mô hình học máy (Ensemble) đã huấn luyện để dự đoán trên một bức ảnh vệ tinh hoàn toàn mới.
* **Thao tác:**
  * Sửa biến `TIF_DIR` trỏ tới thư mục chứa ảnh vệ tinh kiểm thử (`.tif`).
  * Sửa biến `MODEL_DIR` trỏ tới thư mục chứa các file mô hình vừa được tạo ở Bước 3.
* **Kết quả:** Hệ thống sẽ quét qua bức ảnh mới, phân loại từng điểm ảnh và xuất ra bảng tọa độ dự đoán `submission_df` (chỉ ra chính xác các pixel bị mây che phủ).
* `turning.ipynb`: Đánh giá mô hình, quét và tối ưu ngưỡng phân loại (Decision Threshold).
ình (weights) và tự động quét qua các ảnh kiểm thử để kết xuất DataFrame dự đoán cuối cùng (`submission_df`), tái tạo lại mặt nạ phân đoạn mây trên thực tế.

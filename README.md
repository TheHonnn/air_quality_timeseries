📘 Báo cáo Phân tích Dữ liệu & Quy trình Mô hình hóa
1. Q1: Khám phá & Làm sạch Dữ liệu (Preprocessing & EDA)
### 🔹 Tổng quan Dữ liệu

Phạm vi thời gian: Từ 2013-03-01 đến 2017-02-28.

Tần suất: Hourly (hàng giờ). Dữ liệu liên tục, đảm bảo tính chất chuỗi thời gian.

Biến mục tiêu: PM2.5 – nồng độ bụi mịn trong không khí.

Tính dừng (Stationarity):

Kiểm định ADF (Augmented Dickey-Fuller) cho kết quả p-value < 0.05.

Điều này cho thấy chuỗi PM2.5 có tính dừng về mặt thống kê.

Kết luận:

Về lý thuyết, có thể chọn d = 0 cho ARIMA.

Tuy nhiên, do chuỗi vẫn thể hiện mùa vụ và dao động mạnh theo thời gian, việc cân nhắc sai phân (differencing) vẫn cần được xem xét trong thực nghiệm.

### 🔹 Phân tích dữ liệu thiếu (Missing Values)

Dữ liệu bị thiếu ở nhiều nhóm biến:

Nhóm khí tượng (TEMP, PRES, DEWP) thiếu rất ít (≈ 0.1%).

Nhóm ô nhiễm (PM2.5, CO, NO2) thiếu nhiều hơn (≈ 2–5%).

Biểu đồ heatmap missing values cho thấy:

Dữ liệu thiếu thường xuất hiện theo từng đoạn thời gian liên tục (chunks).

Gợi ý nguyên nhân có thể do bảo trì trạm quan trắc hoặc lỗi cảm biến tạm thời.

### ⭐ Insight quan trọng: Tại sao thiếu PM2.5 là đáng lo nhất?

PM2.5 là biến mục tiêu của bài toán.

Các mô hình chuỗi thời gian (như ARIMA) hoạt động dựa trên cơ chế tự hồi quy (Auto-Regressive):

Giá trị hiện tại 
𝑦
𝑡
y
t
	​

 phụ thuộc vào các giá trị quá khứ 
𝑦
𝑡
−
1
,
𝑦
𝑡
−
2
,
…
y
t−1
	​

,y
t−2
	​

,…

Nếu PM2.5 bị thiếu:

Chuỗi thời gian bị đứt đoạn

Mô hình mất thông tin lịch sử cần thiết

Khả năng dự báo liên tục bị suy giảm

➡️ Vì vậy, thiếu biến mục tiêu (PM2.5) nguy hiểm hơn thiếu biến đầu vào như TEMP hay WSPM.

2. Q2: Đánh giá Baseline Hồi quy (Regression Model)

Mô hình baseline sử dụng hồi quy (Linear / Random Forest).

Đặc trưng được xây dựng thông qua Feature Engineering từ chuỗi thời gian:

Lag features

Time-based features (giờ, ngày)

### 🔹 Giải thích kỹ thuật
1. Tại sao Lag 24h lại quan trọng?

PM2.5 chịu ảnh hưởng mạnh bởi:

Nhịp sinh hoạt con người

Chu kỳ tự nhiên ngày – đêm

Ví dụ:

Nồng độ PM2.5 lúc 8h sáng hôm nay thường tương đồng với 8h sáng hôm qua.

Biến lag_24 giúp mô hình nắm bắt được tính mùa vụ theo ngày (Daily Seasonality).

➡️ Đây là một trong những lag quan trọng nhất trong bài toán.

2. Tại sao phải chia Train/Test theo Cutoff thời gian?

Dữ liệu chuỗi thời gian có thứ tự tự nhiên nghiêm ngặt.

Nếu dùng random_split:

Mô hình có thể dùng dữ liệu tương lai để dự đoán quá khứ.

Gây ra Data Leakage (rò rỉ dữ liệu).

Giải pháp đúng:

Cắt dữ liệu theo mốc thời gian (ví dụ: 2017-01-01)

Dữ liệu quá khứ → huấn luyện

Dữ liệu tương lai → kiểm thử

3. Phân biệt RMSE và MAE

MAE (Mean Absolute Error):

Sai số tuyệt đối trung bình

Phản ánh mức sai lệch thông thường hàng ngày

RMSE (Root Mean Squared Error):

Sai số bình phương trung bình

Phạt rất nặng các sai số lớn

Ý nghĩa thực tế:

Nếu RMSE > MAE, điều đó cho thấy mô hình dự báo kém tại các thời điểm có đỉnh ô nhiễm (spikes/outliers).

Nếu mục tiêu là cảnh báo các đợt ô nhiễm nguy hiểm, cần đặc biệt quan tâm đến RMSE.

3. Q3: Quy trình quyết định tham số ARIMA (p, d, q)
### 🔹 Bước 1: Xác định bậc sai phân (d)

Kiểm định ADF test cho thấy chuỗi có tính dừng (p-value < 0.05).

Do đó:

Về lý thuyết, có thể chọn d = 0.

Tuy nhiên, do chuỗi vẫn thể hiện xu hướng và mùa vụ, việc thử nghiệm d = 1 vẫn được cân nhắc để cải thiện mô hình.

### 🔹 Bước 2: Ước lượng p và q bằng ACF / PACF

PACF (Partial Autocorrelation Function):

Gợi ý bậc tự hồi quy p

ACF (Autocorrelation Function):

Gợi ý bậc trung bình trượt q

Việc quan sát điểm cắt và tốc độ suy giảm của các đồ thị này giúp lựa chọn tập giá trị (p, q) ban đầu.

### 🔹 Bước 3: Lựa chọn mô hình tối ưu

Thử nghiệm các tổ hợp (p, d, q) trong phạm vi nhỏ.

Sử dụng AIC (Akaike Information Criterion) để so sánh.

➡️ Mô hình có AIC thấp thể hiện sự cân bằng giữa:

Độ phù hợp dữ liệu

Độ phức tạp mô hình
→ Tránh overfitting.

### 🔹 Bước 4: Kiểm tra phần dư (Residual Diagnostics)

Residual được kỳ vọng:

Dao động quanh 0

Không còn xu hướng hay chu kỳ rõ rệt

Trong thực nghiệm:

Residual chưa hoàn toàn là white noise

Gợi ý rằng mô hình vẫn còn hạn chế

➡️ Đây là cơ sở để đề xuất các hướng mở rộng trong tương lai.

Kết luận

PM2.5 là chuỗi thời gian có tính dừng về mặt thống kê nhưng vẫn thể hiện mùa vụ và biến động mạnh.

Regression với lag features cung cấp baseline hợp lý.

ARIMA khai thác tốt cấu trúc tự tương quan cho dự báo ngắn hạn.

Việc hiểu đúng dữ liệu và giữ nguyên thứ tự thời gian đóng vai trò quan trọng hơn việc sử dụng mô hình phức tạp.

# Dự đoán PM2.5 theo giờ tại Bắc Kinh: Regression vs ARIMA cho Cảnh báo Ngắn hạn



## Mục lục
1. [Giới thiệu bài toán](#1-giới-thiệu-bài-toán)
2. [Thiết lập pipeline và cấu hình tham số](#2-thiết-lập-pipeline-và-cấu-hình-tham-số)
3. [Trực quan hóa và diễn giải dữ liệu](#3-trực-quan-hóa-và-diễn-giải-dữ-liệu)
4. [So sánh Regression và ARIMA](#4-so-sánh-regression-và-arima)
5. [Insight và khuyến nghị](#5-insight-và-khuyến-nghị)
6. [Trả lời Q1–Q3](#6-trả-lời-q1q3)
7. [Kết luận](#7-kết-luận)

---

## 1. Giới thiệu bài toán
PM2.5 là chỉ số ô nhiễm không khí ảnh hưởng trực tiếp đến sức khỏe cộng đồng, đặc biệt tại các đô thị lớn như Bắc Kinh. Mục tiêu của dự án này là dự báo PM2.5 theo giờ (**horizon = 1**) bằng hai cách tiếp cận:
* **Regression baseline:** Sử dụng đặc trưng thời gian và độ trễ (lag features).
* **ARIMA:** Mô hình chuỗi thời gian đơn biến truyền thống.

---

## 2. Thiết lập pipeline và cấu hình tham số
Pipeline được thiết kế theo luồng chặt chẽ: `EDA` $\rightarrow$ `Regression` $\rightarrow$ `ARIMA`.

**Cấu hình tham số chính:**
* **STATION:** Aotizhongxin
* **CUTOFF:** 2017-01-01 (Chia dữ liệu Train/Test)
* **HORIZON:** 1 (Dự báo trước 1 giờ)
* **LAG_HOURS:** [1, 3, 24]

> **Lưu ý quan trọng:** Việc chia train/test theo thời gian (Time-series split) giúp tránh hiện tượng **leakage** – mô hình “nhìn trộm tương lai”.

---

## 3. Trực quan hóa và diễn giải dữ liệu

### 3.1. Toàn cảnh biến động PM2.5
![Hình 1: PM2.5 toàn giai đoạn 2013–2017](./Images/anh1.png)
* Dữ liệu dao động mạnh với nhiều đỉnh cao bất thường (**spike**).
* Dữ liệu có đuôi phải dài, không phân phối chuẩn. Các spike này là rủi ro sức khỏe cực lớn cần dự báo chính xác.

### 3.2. Chu kỳ và đặc điểm ngắn hạn
![Hình 2: PM2.5 phóng to 1–2 tháng](./Images/anh4.png)
* PM2.5 có dao động theo ngày rõ rệt. Mức ô nhiễm thường cao hơn vào ban đêm và sáng sớm.
* **Lag 24h** là đặc trưng sống còn để mô hình không bị lệch chu kỳ sinh hoạt đô thị.

### 3.3. Phân tích tự tương quan
![Hình 3: ACF / PACF của PM2.5](./Images/anh2.png)
* **ACF:** Tự tương quan mạnh ở lag 1, 2 và lặp lại quanh lag 24.
* **PACF:** Giảm dần sau vài lag đầu, gợi ý giá trị $p$ nhỏ trong mô hình ARIMA.

### 3.4. Kết quả dự báo ARIMA
![Hình 4: Forecast vs Actual (ARIMA)](./Images/anh9.png)
* ARIMA bám sát xu hướng nhưng có xu hướng "làm mượt" (smooth) các đỉnh. Điều này khiến sai số RMSE tăng cao tại các điểm spike.

---

## 4. So sánh Regression và ARIMA

| Tiêu chí | Regression Baseline | ARIMA |
| :--- | :--- | :--- |
| **Hiệu suất (H=1)** | Thường tốt hơn (MAE/RMSE thấp hơn) | Trung bình |
| **Phản ứng với Spike** | Nhanh, nhạy bén nhờ lag gần | Chậm, có xu hướng dự báo thấp hơn thực tế |
| **Khả năng mở rộng** | Dễ thêm biến thời tiết, giao thông | Khó (cần chuyển sang SARIMAX) |
| **Độ phức tạp** | Thấp, chạy nhanh | Cao, cần kiểm định tính dừng |

---

## 5. Insight và khuyến nghị
1.  **Cập nhật theo giờ:** Do PM2.5 có spike ngắn hạn, hệ thống cần dự báo real-time.
2.  **Chính sách:** Tập trung hạn chế giao thông vào các khung giờ cao điểm đã được nhận diện qua chu kỳ 24h.
3.  **Đánh giá:** Ưu tiên chỉ số **RMSE** khi đánh giá vì nó phản ánh tốt hơn rủi ro tại các đỉnh ô nhiễm cực đoan.

---

## 6. Trả lời Q1 – Q3

* **Q1 (Hiểu dữ liệu):** Tự tương quan mạnh + Chu kỳ ngày + Nhiều spike. Thiếu dữ liệu tại biến mục tiêu PM2.5 là vấn đề nghiêm trọng nhất cần xử lý.
* **Q2 (Regression):** Lag 24h phản ánh chu kỳ sinh hoạt. Sử dụng Cutoff là bắt buộc để đảm bảo tính khách quan của mô hình chuỗi thời gian.
* **Q3 (ARIMA):** Quy trình chuẩn: Quan sát $\rightarrow$ Kiểm định dừng $\rightarrow$ ACF/PACF $\rightarrow$ Chọn $p,d,q$ qua AIC/BIC $\rightarrow$ Chẩn đoán phần dư.

---

## 7. Kết luận
Không có mô hình “tốt nhất cho mọi trường hợp”. Tuy nhiên, với bài toán **cảnh báo sớm ngắn hạn (1 giờ)**, **Regression Baseline** tỏ ra hiệu quả, thực tế và dễ triển khai hơn. ARIMA sẽ phát huy giá trị tốt hơn trong việc phân tích xu hướng dài hạn.

---
*Dữ liệu phân tích từ trạm Aotizhongxin, Bắc Kinh.*
**Tác giả:**
Bùi Thế Hoàng
Nguyễn Sỹ Quang Huy
Nguyễn Thế Hạnh

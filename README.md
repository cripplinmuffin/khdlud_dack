Môn **Khoa học dữ liệu và Ứng dụng - 17/21**  
Học kì I, 2020 - 2021

---

# ĐỒ ÁN CUỐI KÌ

### Dự đoán giá cổ phiếu

---

## 1. Thông tin nhóm 

| Tên  |MSSV|
|-|:-:|
| Trần Minh Trí |1712834|
| Nguyễn Nhật Trường |1712852|

## 2. Nội dung đồ án

#### Giới thiệu đồ án

* Dự đoán giá cổ phiếu dựa theo pattern giá trong quá khứ.
* Trả lời được câu hỏi trên sẽ giúp người chơi cổ phiếu quyết định mua hay bán một loại cổ phiếu nào đó.

#### Nguồn dữ liệu

* Trang web [CafeF](https://s.cafef.vn/) là nguồn sử dụng để thu thập dữ liệu. Từng loại cổ phiếu được sử dụng sẽ được thu thập thông qua trang tìm kiếm của cổ phiếu đó, cụ thể: [BHV](https://s.cafef.vn/Lich-su-giao-dich-BVH-1.chn?fbclid=IwAR0e98txe3qOw8SP_cTAVxXqeTN2CnuAiOnnLMzUXovyH-zJRZXVNBWU2sg).

#### Thu thập dữ liệu

* Dữ liệu được thu thập bằng phương thức parse HTML sử dụng `selenium`, đến ngày 17/12/2020.

#### Khám phá dữ liệu

* Có 5 cột:
  * `Date`: ngày.
  * `Open`: giá mở cửa.
  * `High`: giá cao nhất.
  * `Low`: giá thấp nhất.
  * `Close`: giá đóng cửa.
* Dữ liệu (chưa tiền xử lý), sau khi đưa cột `Date` thành index, có 2868 dòng và 4 cột, với cả 4 cột đều có kiểu dữ liệu `float`:

![image](./img/data_1.PNG)

![image](./img/data_2.PNG)

#### Các vấn đề sau khi thu thập dữ liệu

* Dữ liệu thu thập được là các dòng theo từng ngày, tuy nhiên, có một số ngày bị thiếu do website không cập nhật giá cổ phiếu vào các ngày đó. Ta cần phải tiền xử lý dữ liệu để "fill" các giá trị còn thiếu ở các ngày đó.

---

## 3. Chi tiết đồ án

### 3.1 Tiền xử lý dữ liệu

* **Cách xử lý**: Thêm những điểm dữ liệu thiếu bằng đoạn giữa với 2 đầu là 2 điểm quan sát được. (VD: [1, nan, nan, 7] -> [1, 3, 5, 7])
* Sau khi tiền xử lý, dữ liệu mới gồm có 4193 dòng và 4 cột:

![image](./img/data_3.PNG)

### 3.2 Trực quan hóa dữ liệu

* Ta có biểu đồ giá đóng cho cổ phiếu BVH như sau:
![image](./img/vis_1.png)
* Biểu đồ dưới đây chỉ thể hiện giá trong 365 ngày cuối để dễ nhìn hơn số liệu thiếu được thêm vào:
![image](./img/vis_2.png)
* Với bộ dữ liệu được xử lý đầy đủ, ta sẽ thực hiện phân tích thành phần chuỗi thời gian (**Time-series decomposition**) cho dãy giá đóng. Quá trình này cho phép chuỗi thời gian được thể hiện qua 3 đặc trưng chính là **Trend**, **Seasonality** và **Noise**:
![image](./img/vis_3.png)
* Ta có nhận xét: chuỗi thời gian hoàn toàn không có tính **seasonality** hay có xu hướng - **trend** nào rõ rệt. Hơn nữa, xét về **noise**, dữ liệu mang tính khá ngẫu nhiên.

### 3.2 Mô hình hóa

* **Tách dữ liệu**: Dữ liệu ban đầu được chia thành 3 tập: train, validation, test với tỉ lệ (gần đúng) 70% - 15% - 15%.
* **Chuẩn hóa dữ liệu**: Nhóm sử dụng MinMaxScaler để chuẩn hóa dữ liệu về `range(0, 1)`.
* **Chuyển dữ liệu về các timestep sequences**:
    * Tập X (input) chứa dữ liệu giá đóng cửa *step* ngày trước ngày *n* cần dự đoán.
    * Tập Y (output) chứa dữ liệu giá đóng cửa 1 ngày *n* cần dự đoán.
    * Biến **lag** là khoảng cách giữa **ngày dự đoán** đến **ngày quan sát được gần nhất - 1**. VD:
        * lag=0 -> Y(n) = f(O(n-1), O(n-2),...)
        * lag=2 -> Y(n) = f(O(n-3), O(n-4),...)

### 3.3 Train model

* Nhóm sử dụng **mô hình LSTM** của thư viện `keras`,  sau đó, thực hiện validation với các tham số `step`, `lag` và `batch_size` (log khi train được lưu ở file "model_log.txt").
* Kết quả chọn ra các model tốt nhất cho từng lag:

![image](./img/model_1.PNG)

* Trực quan hóa các model được chọn trên tập validation:
![image](./img/lag_0.png)

![image](./img/lag_1.png)

![image](./img/lag_3.png)

### 3.4 Kết quả

* Kết quả cho thấy được độ lỗi của `lag = 1, 3` cao hơn nhiều so với `lag = 0`, nhóm quyết định chỉ chọn model tốt nhất `lag = 0` làm model cuối cùng.
* Độ lỗi của model cuối trên tập test:

![image](./img/err_1.png)

![image](./img/err_2.png)

![image](./img/err_3.png)

![image](./img/err_4.png)

### 3.5 Dự đoán

* Nhóm thử dự đoán giá đóng cửa cho 7 ngày tiếp theo, với mỗi ngày dự đoán được thêm vào chuỗi quan sát để dự đoán cho ngày tiếp theo:
    * Y(n) = f(O(n-1), O(n-2),...)
    * Y(n+1) = f(Y(n), O(n-1), O(n-2),...)
* Kết quả thu được như sau:

![image](./img/predict_1.png)

## 4. Đánh giá

### 4.1 Phân công

| Tên  |MSSV|Công việc thực hiện|
|-|:-:|-:|
| Trần Minh Trí |1712834| Mô hình hóa, huấn luyện, dự đoán |
| Nguyễn Nhật Trường |1712852| Thu thập, khám phá , phân tích , tiền xử lí dữ liệu + Báo cáo |

### 4.2 Đánh giá đồ án

* Trong quá trình làm đồ án, các thành viên hoạt động nhóm tốt và hỗ trợ nhau để hoàn thành phần của mình. Một số khó khăn gặp phải như việc đổi cách thu thập dữ liệu từ API sang parse HTML hay cân nhắc cách đánh giá model hợp lý. Tuy nhiên, nhóm đã hết sức cố gắng giải quyết các vấn đề trên và hoàn thành đồ án với kết quả gần như mong đợi.

## 5. Hướng dẫn chạy file Notebook

* Phần thu thập dữ liệu sử dụng `selenium`, cần tải về [chromedriver](https://chromedriver.chromium.org/downloads) phù hợp dựa theo phiên bản trình duyệt Google Chrome của máy.
* Dữ liệu đã được lưu ở trong thư mục "csv".
* Ở phần huấn luyện các model, **do load nhiều model rất lâu** nên có lưu ý sau:

![image](./img/note_1.PNG)

* Các model đã được lưu lại ở thư mục "models" để rút ngắn thời gian chạy. Thực hiện đổi giá trị biến `known` để huấn luyện lại các model (mất khoảng 1h).
* Nếu bỏ qua, file notebook chạy bình thường thông qua `Kernel - Restart & Run All`.

## 6. Nguồn tham khảo

[1] https://insights.magestore.com/posts/giai-thuat-time-series-forecasting

[2] https://www.datacamp.com/community/tutorials/lstm-python-stock-market

[3] https://randerson112358.medium.com/stock-price-prediction-using-python-machine-learning-e82a039ac2bb

[4] https://viblo.asia/p/lam-quen-voi-keras-gGJ59mxJ5X2

[5] https://www.tensorflow.org/api_docs/python/tf/keras/Sequential

[6] https://www.tensorflow.org/api_docs/python/tf/keras/optimizers

[7] https://www.tensorflow.org/api_docs/python/tf/keras/layers/LSTM

[8] https://www.tensorflow.org/api_docs/python/tf/keras/layers/Dense

[9] https://viblo.asia/p/optimizer-hieu-sau-ve-cac-thuat-toan-toi-uu-gdsgdadam-Qbq5QQ9E5D8

[10] https://www.phamduytung.com/blog/2018-10-02-understanding-epoch-batchsize-iterations/

[11] https://otexts.com/fpp2/prediction-intervals.html
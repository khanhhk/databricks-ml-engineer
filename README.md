# Databricks Certified Machine Learning

## Databricks Certified Machine Learning Associate 

### Thông tin về kỳ thi

- Số lượng câu hỏi: 48 câu trắc nghiệm tính điểm, bao gồm dạng một lựa chọn hoặc nhiều lựa chọn
- Thời gian làm bài: 90 phút
- Lệ phí đăng ký: 200 USD
- Thời hạn hiệu lực: 2 năm

### Đề cương kỳ thi

#### Phần 1: Databricks Machine Learning

- Xác định best practices của một chiến lược MLOps
- Xác định các lợi thế của việc sử dụng ML runtimes
- Xác định cách AutoML hỗ trợ việc lựa chọn mô hình và đặc trưng
- Xác định các lợi ích mà AutoML mang lại cho quy trình phát triển mô hình
- Xác định lợi ích của việc tạo bảng feature store ở cấp tài khoản (account level) trong Unity Catalog trên Databricks so với cấp workspace (workspace level)
- Tạo bảng feature store trong Unity Catalog
- Ghi dữ liệu vào bảng feature store
- Huấn luyện mô hình với các đặc trưng lấy từ bảng feature store
- Chấm điểm mô hình bằng các đặc trưng lấy từ bảng feature store
- Mô tả sự khác biệt giữa bảng đặc trưng online và offline
- Xác định best run khi sử dụng MLflow Client API
- Ghi log thủ công các metric, artifact và model trong một MLflow Run
- Xác định thông tin có sẵn trong MLflow UI
- Đăng ký (Register) mô hình bằng MLflow Client API trong Unity Catalog registry
- Xác định lợi ích của việc đăng ký mô hình trong Unity Catalog registry so với workspace registry
- Xác định các tình huống mà việc promote code phù hợp hơn việc promote model và ngược lại
- Thiết lập hoặc gỡ bỏ tag cho một mô hình
- Promote một challenger model thành champion model bằng aliases

#### Phần 2: Data Processing

- Tính toán thống kê tóm tắt (summary statistics) trên Spark DataFrame bằng `.summary()` hoặc dbutils data summaries
- Loại bỏ outlier khỏi Spark DataFrame dựa trên độ lệch chuẩn hoặc IQR
- Tạo biểu đồ trực quan hóa cho đặc trưng phân loại hoặc liên tục
- So sánh hai đặc trưng phân loại hoặc hai đặc trưng liên tục bằng phương pháp phù hợp
- So sánh và đối chiếu việc điền giá trị thiếu bằng mean, median hoặc mode
- Điền giá trị thiếu bằng mode, mean hoặc median
- Sử dụng one-hot encoding cho các đặc trưng phân loại
- Xác định và giải thích các loại mô hình hoặc bộ dữ liệu mà one-hot encoding phù hợp hoặc không phù hợp
- Xác định các tình huống mà biến đổi log scale là phù hợp

#### Phần 3: Model Development

- Sử dụng nền tảng kiến thức machine learning để chọn thuật toán phù hợp cho một kịch bản cụ thể
- Xác định các phương pháp giảm mất cân bằng dữ liệu trong dữ liệu huấn luyện
- So sánh estimators và transformers
- Xây dựng training pipeline
- Sử dụng phép toán `fmin` của Hyperopt để tinh chỉnh siêu tham số cho mô hình
- Thực hiện random search, grid search hoặc Bayesian search như một phương pháp tuning hyperparameters
- Song song hóa các mô hình chạy trên một node để tuning hyperparameters
- Mô tả lợi ích và hạn chế của việc dùng cross-validation thay cho train-validation split
- Thực hiện cross-validation như một phần của quá trình fitting mô hình
- Xác định số lượng mô hình được huấn luyện khi kết hợp grid search với cross-validation
- Sử dụng các metric phổ biến cho bài toán phân loại: F1, Log Loss, ROC/AUC, v.v.
- Sử dụng các metric phổ biến cho bài toán hồi quy: RMSE, MAE, R-squared, v.v.
- Chọn metric phù hợp nhất cho mục tiêu của một kịch bản cụ thể
- Xác định sự cần thiết phải lấy lũy thừa ngược cho các biến đích đã được log-transform trước khi tính metric đánh giá hoặc diễn giải dự đoán
- Đánh giá tác động của độ phức tạp mô hình và sự đánh đổi bias-variance đối với hiệu năng mô hình

#### Phần 4: Model Development

- Xác định sự khác biệt và ưu điểm của các phương thức serving như batch, real-time và streaming
- Triển khai mô hình tùy chỉnh lên model endpoint
- Sử dụng pandas để thực hiện batch inference
- Xác định cách streaming inference được thực hiện với Delta Live Tables
- Triển khai và truy vấn mô hình cho realtime inference
- Phân chia dữ liệu giữa các endpoint để phục vụ realtime inference


## Databricks Certified Machine Learning Professional

### Thông tin về kỳ thi

- Số lượng câu hỏi: 59 câu trắc nghiệm tính điểm
- Thời gian làm bài: 120 phút
- Lệ phí đăng ký: 200 USD
- Thời hạn hiệu lực: 2 năm

### Đề cương kỳ thi

#### Phần 1: Model Development

##### Using Spark ML

- Xác định khi nào nên dùng SparkML dựa trên dữ liệu, mô hình và yêu cầu của bài toán
- Xây dựng pipeline machine learning bằng SparkML
- Áp dụng estimator và/hoặc transformer phù hợp theo từng trường hợp sử dụng
- Tinh chỉnh mô hình SparkML bằng MLlib
- Đánh giá (Evaluate) mô hình SparkML
- Chấm điểm mô hình SparkML cho use case batch hoặc streaming
- Lựa chọn giữa mô hình SparkML hoặc mô hình single-node cho suy luận: batch, real-time hoặc streaming

##### Scaling and Tuning

- Mở rộng pipeline huấn luyện phân tán bằng SparkML và pandas Function APIs/UDFs
- Thực hiện tinh chỉnh siêu tham số phân tán bằng Optuna và tích hợp với MLflow
- Thực hiện tinh chỉnh siêu tham số phân tán bằng Ray
- Đánh giá sự đánh đổi giữa vertical scaling và horizontal scaling cho workload machine learning trong môi trường Databricks 
- Đánh giá và chọn chiến lược song song hóa phù hợp, gồm model parallelism và data parallelism, cho huấn luyện ML quy mô lớn
- So sánh Ray và Spark trong việc phân phối các workload huấn luyện ML
- Sử dụng Pandas Function API để song song hóa huấn luyện mô hình theo từng nhóm và thực hiện suy luận

##### Advanced MLflow Usage

- Sử dụng nested runs trong MLflow để theo dõi các thí nghiệm phức tạp
- Ghi log custom metrics, parameters và artifacts bằng lập trình trong MLflow để theo dõi các quy trình thử nghiệm nâng cao
- Tạo custom model objects bằng cách sử dụng real-time feature engineering

##### Advanced Feature Store Concepts

- Bảo đảm tính đúng đắn theo thời điểm thực (point-in-time) của feature lookups để tránh rò rỉ dữ liệu (data leakage) khi huấn luyện và suy luận
- Xây dựng pipeline tự động cho việc tính toán đặc trưng bằng FeatureEngineering Client
- Cấu hình online tables cho các ứng dụng độ trễ thấp (low-latency) bằng Databricks SDK
- Thiết kế giải pháp có khả năng mở rộng để nạp (ingest) và xử lý dữ liệu streaming nhằm tạo đặc trưng theo thời gian thực
- Phát triển on-demand features bằng feature serving để sử dụng nhất quán (consistent) giữa môi trường huấn luyện và production

#### Phần 2: MLOps

##### Model Lifecycle Management

- Mô tả và triển khai các thành phần kiến trúc của pipeline vòng đời mô hình dùng để quản lý việc chuyển đổi môi trường trong chiến lược triển khai bằng code
- Ánh xạ các tính năng của Databricks vào các hoạt động trong quy trình quản lý vòng đời mô hình

##### Validation Testing

- Triển khai unit tests cho các hàm riêng lẻ trong Databricks notebooks để bảo đảm chúng tạo ra đầu ra mong đợi với các đầu vào cụ thể
- Xác định các loại kiểm thử được thực hiện, như unit và integration, trong các giai đoạn môi trường khác nhau như dev, test, prod
- Thiết kế integration test cho hệ thống machine learning có tích hợp các pipeline phổ biến: feature engineering, training, evaluation, deployment và inference
- So sánh lợi ích và thách thức của các cách tổ chức hàm và unit tests

##### Environment Architectures

- Thiết kế và triển khai các môi trường Databricks có khả năng mở rộng cho dự án machine learning theo best practices
- Định nghĩa và cấu hình Databricks ML assets bằng DABs (Databricks Asset Bundles): model serving endpoints, MLflow experiments, ML registered models

##### Automated Retraining

- Triển khai các quy trình huấn luyện lại tự động có thể được kích hoạt bởi phát hiện data drift hoặc cảnh báo suy giảm hiệu năng mô hình
- Phát triển chiến lược chọn ra các mô hình có hiệu năng tốt nhất trong quá trình huấn luyện lại tự động

##### Drift Detection and Lakehouse Monitoring

- Áp dụng các kiểm định thống kê (statistical tests) từ bảng drift metrics trong Lakehouse Monitoring để phát hiện drift trong dữ liệu số và dữ liệu phân loại, đồng thời đánh giá mức độ ý nghĩa của các thay đổi quan sát được
- Xác định loại bảng dữ liệu và tính năng Lakehouse Monitoring phù hợp để giải quyết nhu cầu use case, đồng thời giải thích lý do
- Xây dựng monitor cho snapshot table, time series table hoặc inference table bằng Lakehouse Monitoring
- Xác định các thành phần chính của các pipeline giám sát phổ biến: logging, drift detection, model performance, model health, v.v.
- Thiết kế và cấu hình cơ chế cảnh báo để thông báo cho các bên liên quan khi drift metrics vượt quá ngưỡng định sẵn
- Phát hiện data drift bằng cách so sánh phân phối dữ liệu hiện tại với baseline đã biết hoặc giữa các cửa sổ thời gian liên tiếp
- Đánh giá xu hướng hiệu năng mô hình theo thời gian bằng inference table
- Định nghĩa custom metrics trong các bảng metrics của Lakehouse Monitoring
- Đánh giá metrics dựa trên các mức độ chi tiết dữ liệu khác nhau (different data granularities) và feature slicing
- Giám sát endpoint health bằng cách theo dõi các hạ tầng metrics như latency, request rate, error rate, CPU usage và memory usage

#### Phần 3: Model Deployment

##### Deployment Strategies

- So sánh các chiến lược triển khai, chẳng hạn blue-green và canary, đồng thời đánh giá mức độ phù hợp của chúng đối với các ứng dụng có lưu lượng lớn
- Triển khai chiến lược rollout mô hình bằng Databricks Model Serving

##### Custom Model Serving

- Đăng ký custom PyFunc model và ghi log custom artifacts trong Unity Catalog
- Truy vấn custom models qua REST API hoặc MLflow Deployments SDK
- Triển khai custom model objects bằng MLflow deployments SDK, REST API hoặc giao diện người dùng

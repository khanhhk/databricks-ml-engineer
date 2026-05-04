# Databricks Certified Machine Learning Associate — Tài liệu ôn thi

> Tài liệu này được tổng hợp từ tài liệu chính thức của Databricks, dịch sang tiếng Việt, giữ nguyên các thuật ngữ kỹ thuật tiếng Anh. Các mục được đánh dấu ⚠️ là những điểm quan trọng, thường xuất hiện trong đề thi.

---

## Thông tin kỳ thi

| Hạng mục | Chi tiết |
|----------|----------|
| Số câu hỏi | 48 câu trắc nghiệm (một hoặc nhiều lựa chọn) |
| Thời gian | 90 phút |
| Lệ phí | 200 USD |
| Hiệu lực | 2 năm |
| Hình thức | Online proctored hoặc test center |

---

## Phần 1: Databricks Machine Learning

### 1.1 MLOps Strategy — Chiến lược tốt nhất

MLOps (Machine Learning Operations) là tập hợp các thực hành kết hợp Machine Learning với vòng đời phát triển phần mềm nhằm tự động hóa và cải thiện chất lượng triển khai mô hình vào production.

**Ba môi trường chuẩn trong MLOps:**

| Môi trường | Mục tiêu | Quyền truy cập |
|-----------|----------|----------------|
| Development (Dev) | Thử nghiệm, phát triển model | Read-write với dev catalog, read-only với prod data |
| Staging | Kiểm thử code trước khi lên production | Chạy unit test và integration test |
| Production (Prod) | Triển khai và vận hành thực tế | ML engineers quản lý |

**Nguyên tắc cốt lõi:**
- Sử dụng **Git** để version control cho tất cả code pipeline
- Dùng **Unity Catalog** để quản lý governance và lineage cho model và data
- Dùng **MLflow** để tracking experiment và quản lý model lifecycle
- **Databricks Workflows** để orchestrate các multi-task pipeline

> ⚠️ **Lưu ý thi:** Databricks khuyến nghị **promote code** (thay vì promote model) từ môi trường này sang môi trường khác. Lý do: đảm bảo tất cả code đều trải qua code review và integration testing, và model production được training trên production code. Chỉ promote model (không phải code) khi training rất tốn kém hoặc nhóm không có quy trình CI/CD.

---

### 1.2 ML Runtime — Databricks Runtime for Machine Learning

**Databricks Runtime for Machine Learning (ML)** là môi trường compute được cấu hình sẵn, tự động tạo resource với các thư viện machine learning và deep learning phổ biến được cài đặt trước.

**Đặc điểm chính:**
- Pre-installed các thư viện ML/DL phổ biến (PyTorch, TensorFlow, XGBoost, scikit-learn, v.v.)
- Cập nhật thư viện theo từng phiên bản runtime
- Hỗ trợ GPU workloads (chọn GPU-enabled instance types)

**Cách tạo compute với ML Runtime:**
- Chọn checkbox **"Machine learning"** khi tạo compute
- Access mode mặc định là **Dedicated** (bắt buộc để truy cập Unity Catalog)

**Photon Integration:**
- Có từ Runtime 15.2 ML trở lên
- Tăng tốc cho: Spark SQL, DataFrames, feature engineering, XGBoost4j
- **Không** cải thiện các package Python thuần túy như PyTorch hoặc TensorFlow

**Graviton Support:**
- Có từ Runtime 15.4 LTS ML trở lên (AWS Graviton instances)
- Hiệu suất/chi phí tốt hơn so với EC2 instance thông thường

**Hỗ trợ training:**
- AutoML, Foundation Model Fine-tuning, Deep Learning (distributed), Hyperparameter Optimization (Optuna)

> ⚠️ **Lưu ý thi:** Lợi thế của ML Runtime so với Standard Runtime là: (1) đã có sẵn các thư viện ML phổ biến, (2) tự động cấu hình Dedicated access mode cho Unity Catalog, (3) được tối ưu và test cho workload ML. Không cần cài thêm thư viện thủ công.

---

### 1.3 AutoML — Tự động hóa lựa chọn mô hình và đặc trưng

**Databricks AutoML** tự động hóa quy trình áp dụng machine learning vào dataset bằng cách tự động tìm thuật toán và cấu hình hyperparameter tốt nhất.

**Quy trình 4 bước của AutoML:**

1. **Data Preparation** — Tự động làm sạch và chuẩn bị dataset
2. **Distributed Training** — Tổ chức training phân tán và tuning hyperparameter trên nhiều thuật toán cùng lúc
3. **Model Evaluation** — Xác định model tốt nhất bằng các evaluation algorithm
4. **Code Generation** — Tạo ra các notebook source code mà bạn có thể xem xét và chỉnh sửa

**Loại bài toán được hỗ trợ:**
- Regression
- Classification
- Time series forecasting

**Thuật toán được hỗ trợ:**

| Bài toán | Thuật toán |
|---------|-----------|
| Classification / Regression | Decision Tree, Random Forest, Logistic/Linear Regression, XGBoost, LightGBM |
| Forecasting | Prophet, Auto-ARIMA, DeepAR |

**Lợi ích của AutoML:**
- Giảm thiểu công việc thủ công trong lựa chọn thuật toán và tuning hyperparameter
- Tạo notebook có thể chỉnh sửa → giúp data scientist học được từ kết quả
- Feature importance thông qua Shapley values (SHAP) — mặc định tắt vì tốn bộ nhớ
- Tích hợp với MLflow để log kết quả

> ⚠️ **Lưu ý thi:** AutoML giúp **lựa chọn model và đặc trưng** vì: (1) thử nhiều thuật toán tự động, (2) sinh ra trial notebooks để xem xét, (3) tích hợp Shapley values cho feature importance. Yêu cầu: package `databricks-automl-runtime`, compute Dedicated hoặc No Isolation Shared.

---

### 1.4 Feature Store — Unity Catalog

**Databricks Feature Store** là trung tâm lưu trữ và quản lý các feature (đặc trưng) dùng trong AI và ML model. Từ Unity Catalog trở đi, feature tables được tích hợp trực tiếp vào Unity Catalog.

#### 1.4.1 Account-level vs Workspace-level Feature Store

> ⚠️ **Lưu ý thi:** Đây là điểm hay ra đề. Hiểu rõ sự khác biệt:

| Tiêu chí | Unity Catalog (Account-level) | Workspace Feature Store (Legacy) |
|---------|-------------------------------|----------------------------------|
| Phạm vi chia sẻ | **Cross-workspace** — dùng được ở nhiều workspace | Chỉ trong 1 workspace |
| Governance | Built-in với Unity Catalog (access control, lineage, audit) | Hạn chế |
| Trạng thái | **Khuyến nghị hiện tại** | Deprecated (sắp bị xóa) |
| Đặt tên | 3 cấp: `<catalog>.<schema>.<table>` | 2 cấp |

**Lợi ích của Unity Catalog Feature Store:**
- **Cross-workspace access**: Feature tables dùng chung giữa nhiều workspace trong cùng account
- **Governance tập trung**: Access control, lineage tracking, audit log tích hợp sẵn
- **Feature discovery**: Tìm kiếm và quản lý qua Catalog Explorer
- **Tagging**: Gán tag để tổ chức và tìm kiếm feature

#### 1.4.2 Tạo bảng Feature Store trong Unity Catalog

Quy trình:
1. Kích hoạt Unity Catalog cho workspace
2. Dùng `FeatureEngineeringClient` (API mới) để tạo feature table
3. Đặt tên theo cú pháp 3 cấp: `catalog.schema.table_name`

#### 1.4.3 Ghi dữ liệu vào Feature Table

- Dùng `FeatureEngineeringClient.write_table()` hoặc `create_table()` để ghi dữ liệu vào feature table
- Hỗ trợ cả batch DataFrame và streaming DataFrame (để tạo streaming feature pipeline)
- DataFrame đầu vào phải có primary key (có thể là composite key)

#### 1.4.4 Training model với Feature Store

- Dùng `FeatureEngineeringClient.create_training_set()` để tạo training dataset
- MLflow tự động theo dõi lineage từ model đến feature table
- Hỗ trợ **point-in-time lookups**: join feature theo đúng thời điểm của observation để tránh data leakage

#### 1.4.5 Scoring model với Feature Store

- Dùng `score_batch()` để thực hiện batch inference
- Model tự động tra cứu giá trị feature mới nhất tại thời điểm inference
- Loại bỏ **training/serving skew** (sự khác biệt giữa feature dùng khi training và khi serving)

#### 1.4.6 Online Store vs Offline Store

> ⚠️ **Lưu ý thi:** Phân biệt rõ hai loại này:

| Tiêu chí | Online Feature Store | Offline Feature Store |
|---------|---------------------|----------------------|
| Mục đích | Real-time inference | Training và batch inference |
| Latency | Millisecond (rất thấp) | Cao hơn (phút tới giờ) |
| Storage | Key-value store tối ưu cho read nhanh | Delta tables |
| Use case | Model serving endpoint, ứng dụng real-time | Tạo training dataset, batch scoring |

---

### 1.5 MLflow — Experiment Tracking và Model Registry

**MLflow** là nền tảng mã nguồn mở lớn nhất cho AI engineering, tích hợp sâu vào Databricks để quản lý toàn bộ vòng đời ML model.

#### 1.5.1 Xác định best run bằng MLflow Client API

Mỗi **MLflow Experiment** chứa nhiều **Run**. Mỗi Run ghi lại:
- **Parameters**: Hyperparameter của model
- **Metrics**: Kết quả đánh giá (accuracy, RMSE, AUC, v.v.)
- **Artifacts**: File output (model file, plot, CSV, v.v.)
- **Tags**: Metadata tùy chỉnh

Để tìm best run, dùng `MlflowClient.search_runs()`:
- Lọc theo experiment ID
- Sắp xếp theo metric (ví dụ: `metrics.val_accuracy DESC`)
- Kết quả là pandas DataFrame

> ⚠️ **Lưu ý thi:** Dùng `MlflowClient` (Python class) để thao tác programmatic với MLflow. Các method thường dùng: `search_runs()`, `get_run()`, `log_metric()`, `log_param()`, `log_artifact()`, `register_model()`.

#### 1.5.2 Log thủ công metric, artifact và model

**Log metric:**
- `mlflow.log_metric("accuracy", 0.92)` — log 1 metric
- `mlflow.log_metrics({"accuracy": 0.92, "loss": 0.08})` — log nhiều metric

**Log parameter:**
- `mlflow.log_param("learning_rate", 0.01)`
- `mlflow.log_params({"lr": 0.01, "epochs": 100})`

**Log artifact:**
- `mlflow.log_artifact("output.csv")` — log 1 file
- `mlflow.log_artifacts("plots/")` — log cả thư mục

**Log model:**
- `mlflow.sklearn.log_model(model, "model")`
- `mlflow.pyfunc.log_model(...)` — cho custom model

**Autologging:** `mlflow.autolog()` — tự động log params, metrics, model cho các framework phổ biến (scikit-learn, XGBoost, PyTorch, v.v.)

#### 1.5.3 Thông tin có trong MLflow UI

MLflow UI cho phép xem:
- Danh sách experiments và runs
- So sánh metrics giữa các runs (parallel coordinates, scatter plot)
- Chi tiết từng run: parameters, metrics, artifacts, tags
- Model artifacts đã được log
- Registered models và các version của chúng
- Lineage từ model đến training data (khi dùng với Unity Catalog)

#### 1.5.4 Đăng ký model bằng MLflow Client API trong Unity Catalog

```
mlflow.register_model(model_uri, "catalog.schema.model_name")
```

- Tự động tạo registered model nếu chưa có
- Mỗi lần register tạo ra một version mới
- Tất cả model version trong Unity Catalog phải có **model signature**

> ⚠️ **Lưu ý thi:** Lợi ích của Unity Catalog Model Registry so với Workspace Registry:
> - **Cross-workspace**: Dùng chung model giữa nhiều workspace
> - **Governance tập trung**: Access control, audit log, lineage
> - **3-level naming**: `catalog.schema.model` thay vì chỉ `model_name`
> - **Không bị giới hạn quota** như Workspace Registry

#### 1.5.5 Unity Catalog Registry vs Workspace Registry

| Tiêu chí | Unity Catalog Registry | Workspace Registry (Legacy) |
|---------|----------------------|-----------------------------|
| Phạm vi | Account-level (cross-workspace) | Workspace-level |
| Governance | Tích hợp Unity Catalog | Độc lập |
| Naming | `catalog.schema.model` | `model_name` |
| Trạng thái | **Khuyến nghị** | Deprecated |

#### 1.5.6 Promote code vs Promote model

> ⚠️ **Lưu ý thi:** Câu hỏi hay xuất hiện về khi nào nên promote code, khi nào nên promote model:

**Promote code phù hợp hơn khi:**
- Nhóm có quy trình CI/CD (unit tests, integration tests)
- Muốn đảm bảo code production được review và test kỹ
- Có thể training lại model nhanh trên production data

**Promote model phù hợp hơn khi:**
- Training rất tốn kém hoặc mất nhiều thời gian (không thể train lại dễ dàng)
- Môi trường single workspace (không có staging riêng biệt)
- Nhóm không có quy trình CI/CD

#### 1.5.7 Thiết lập và gỡ bỏ tag cho model

- Tag là cặp key-value để gán metadata cho registered model hoặc model version
- Ví dụ: `task:classification`, `validation_status:approved`, `team:fraud-detection`
- Dùng `MlflowClient.set_model_tag()` hoặc `set_model_version_tag()`
- Dùng `MlflowClient.delete_model_tag()` để gỡ tag

#### 1.5.8 Promote challenger model thành champion bằng aliases

**Aliases** là tên tham chiếu (mutable) đến một model version cụ thể, không phải stage cố định.

Quy trình:
1. Assign alias "Champion" cho model version đang production
2. Deploy "Challenger" model mới với alias riêng
3. So sánh kết quả Champion vs Challenger
4. Khi Challenger tốt hơn: reassign alias "Champion" sang version của Challenger
5. Inference code dùng alias không cần thay đổi — tự động dùng version mới

Load model theo alias:
```
models:/catalog.schema.model_name@Champion
```

> ⚠️ **Lưu ý thi:** Aliases (`@Champion`, `@Challenger`) thay thế cho **stages** cũ (None → Staging → Production → Archived). Trong Unity Catalog, không còn dùng stages; dùng aliases thay thế.

---

## Phần 2: Data Processing

### 2.1 Tính toán summary statistics trên Spark DataFrame

**Phương thức `.summary()`:**
- Trả về thống kê: count, mean, stddev, min, 25%, 50%, 75%, max cho mỗi cột số
- Cú pháp: `df.summary()` hoặc `df.summary("count", "mean", "stddev")`

**Phương thức `.describe()`:**
- Tương tự nhưng chỉ gồm: count, mean, stddev, min, max (không có percentile)

**dbutils data summaries:**
- Databricks cung cấp visualization tích hợp trong notebook
- Sau khi chạy `display(df)`, có thể xem summary statistics trực quan

> ⚠️ **Lưu ý thi:** `.summary()` có thêm các percentile (25%, 50%, 75%) so với `.describe()`. Dùng `.summary()` khi cần biết phân phối dữ liệu đầy đủ hơn.

---

### 2.2 Loại bỏ outlier

**Phương pháp 1: Standard Deviation (Độ lệch chuẩn)**
- Ngưỡng thường dùng: mean ± 3σ (sigma)
- Giữ lại các giá trị nằm trong khoảng [mean - 3σ, mean + 3σ]
- Phù hợp khi dữ liệu có phân phối xấp xỉ chuẩn (normal distribution)

**Phương pháp 2: IQR (Interquartile Range)**
- IQR = Q3 - Q1 (khoảng tứ phân vị)
- Ngưỡng dưới (lower fence): Q1 - 1.5 × IQR
- Ngưỡng trên (upper fence): Q3 + 1.5 × IQR
- Phù hợp khi dữ liệu có phân phối lệch (skewed distribution)

> ⚠️ **Lưu ý thi:** IQR **robust hơn** với phân phối lệch; Standard Deviation phù hợp hơn với phân phối chuẩn. Trên Spark DataFrame, dùng `.summary()` để lấy Q1, Q3, sau đó filter theo ngưỡng tính được.

---

### 2.3 Trực quan hóa đặc trưng

**Đặc trưng phân loại (Categorical feature):**
- **Bar chart / Count plot**: Hiển thị tần suất (frequency) của từng giá trị
- Dùng để nhận biết class imbalance hoặc giá trị hiếm (rare category)

**Đặc trưng liên tục (Continuous feature):**
- **Histogram**: Phân phối giá trị
- **Box plot**: Phân vị, median, và outlier
- **KDE plot** (Kernel Density Estimation): Đường cong phân phối mượt hơn histogram

**So sánh hai đặc trưng:**

| Kết hợp | Phương pháp phù hợp |
|---------|---------------------|
| Categorical vs Categorical | Stacked bar chart, heatmap (confusion matrix style), chi-square test |
| Continuous vs Continuous | Scatter plot, correlation matrix, Pearson/Spearman correlation |
| Categorical vs Continuous | Box plot theo nhóm, violin plot, ANOVA |

---

### 2.4 Xử lý missing values (Điền giá trị thiếu)

> ⚠️ **Lưu ý thi:** Biết khi nào dùng loại nào:

| Phương pháp | Khi nào dùng | Điểm cần chú ý |
|------------|-------------|----------------|
| **Mean (Trung bình)** | Dữ liệu liên tục, phân phối chuẩn, ít outlier | **Bị ảnh hưởng bởi outlier** |
| **Median (Trung vị)** | Dữ liệu liên tục, **phân phối lệch hoặc có outlier** | Robust với outlier |
| **Mode (Yếu vị)** | Dữ liệu **phân loại (categorical)** hoặc rời rạc | Dùng giá trị xuất hiện nhiều nhất |

**Trên Spark DataFrame:**
- Tính mean/median/mode bằng `.agg()` và `.approxQuantile()`
- Điền bằng `df.fillna({"column": value})`

---

### 2.5 One-hot encoding và Log scale

#### One-hot Encoding

**Khái niệm:** Biến đổi mỗi giá trị phân loại thành một vector nhị phân (0/1) với số chiều bằng số lượng category.

Ví dụ: Cột "Color" có 3 giá trị [Red, Blue, Green]:
- Red → [1, 0, 0]
- Blue → [0, 1, 0]
- Green → [0, 0, 1]

**Khi nào one-hot encoding phù hợp:**
- Đặc trưng phân loại danh nghĩa (nominal) — không có thứ tự tự nhiên
- Thuật toán không giả định thứ tự số (linear models, neural networks, SVM)
- Số lượng category vừa phải (low cardinality)

**Khi nào one-hot encoding KHÔNG phù hợp:**

> ⚠️ **Lưu ý thi:**

| Trường hợp | Lý do |
|-----------|-------|
| Đặc trưng có **high cardinality** (nhiều category, ví dụ: zip code, user ID) | Tạo ra quá nhiều chiều → curse of dimensionality |
| Mô hình **tree-based** (Decision Tree, Random Forest, XGBoost) | Tree models xử lý được categorical trực tiếp, không cần one-hot |
| Dataset **rất lớn** | Tốn bộ nhớ và tính toán vì tạo sparse matrix |
| Đặc trưng có **thứ tự** (ordinal) — ví dụ: low/medium/high | Nên dùng ordinal encoding để giữ thứ tự |

**Trong Spark MLlib:** Dùng `StringIndexer` (label encoding) trước, sau đó `OneHotEncoder`.

#### Log Scale Transformation

**Khi nào log transformation phù hợp:**
- Dữ liệu có phân phối **right-skewed** (lệch phải) — ví dụ: thu nhập, giá nhà, view trên web
- Khoảng giá trị rất rộng (vài đơn vị đến hàng triệu)
- Quan hệ giữa feature và target theo dạng nhân (multiplicative) thay vì cộng (additive)
- Mô hình tuyến tính yêu cầu đặc trưng có phân phối gần chuẩn

**Lưu ý quan trọng:**
- Không áp dụng log cho giá trị <= 0 (phải dùng `log(x + 1)` nếu có 0)
- Khi target variable được log-transform, cần áp dụng `exp()` (lũy thừa ngược) cho predictions trước khi tính metric hoặc diễn giải

> ⚠️ **Lưu ý thi:** Nếu bài toán log-transform biến target (y), phải **exp() kết quả dự đoán** trước khi tính RMSE hoặc MAE. Không làm điều này sẽ dẫn đến metric sai.

---

## Phần 3: Model Development

### 3.1 Chọn thuật toán phù hợp

**Bài toán Classification:**
- **Logistic Regression**: Đơn giản, diễn giải được, phù hợp với dữ liệu linearly separable
- **Decision Tree**: Dễ diễn giải, xử lý non-linear, nhưng dễ overfit
- **Random Forest**: Ensemble của nhiều Decision Trees, ổn định hơn, giảm variance
- **Gradient Boosted Trees (GBT) / XGBoost / LightGBM**: Hiệu năng cao, phù hợp dữ liệu tabular phức tạp
- **SVM (Support Vector Machine)**: Tốt với dữ liệu high-dimensional, nhưng chậm trên large dataset
- **Neural Network**: Tốt với unstructured data (ảnh, văn bản, âm thanh)

**Bài toán Regression:**
- **Linear Regression**: Đơn giản, nhanh, cần quan hệ tuyến tính
- **Decision Tree / Random Forest Regression**: Non-linear, xử lý interaction tốt
- **XGBoost / LightGBM Regression**: Hiệu năng cao
- **Ridge, Lasso, ElasticNet**: Linear với regularization, xử lý multicollinearity

> ⚠️ **Lưu ý thi:** Khi câu hỏi hỏi về **chọn thuật toán cho kịch bản cụ thể**: (1) Dữ liệu nhỏ, cần giải thích → Logistic Regression hoặc Decision Tree; (2) Dữ liệu lớn, cần hiệu năng cao → XGBoost/LightGBM; (3) Unstructured data → Neural Network; (4) Nhiều features, sparse → SVM hoặc regularized linear.

---

### 3.2 Xử lý Class Imbalance (Mất cân bằng dữ liệu)

**Nguyên nhân:** Một class (thường là class dương tính / minority) có số lượng mẫu ít hơn nhiều so với class còn lại.

**Các phương pháp giảm mất cân bằng:**

| Phương pháp | Mô tả |
|------------|-------|
| **Oversampling** (ví dụ: SMOTE) | Tạo thêm mẫu giả cho minority class |
| **Undersampling** | Giảm mẫu của majority class |
| **Class weights** | Gán trọng số cao hơn cho minority class trong loss function |
| **Threshold tuning** | Điều chỉnh ngưỡng phân loại thay vì dùng 0.5 mặc định |
| **Ensemble methods** (BalancedBaggingClassifier) | Kết hợp nhiều model trên balanced subsets |

> ⚠️ **Lưu ý thi:** Metric thích hợp khi có class imbalance là **F1-score**, **AUC-ROC**, **Precision-Recall curve** — **không phải Accuracy** (Accuracy cao nhưng model có thể chỉ đoán toàn majority class).

---

### 3.3 Estimator vs Transformer

Trong Spark MLlib, pipeline được xây dựng từ hai loại component:

| Khái niệm | Mô tả | Phương thức | Ví dụ |
|-----------|-------|------------|-------|
| **Transformer** | Biến đổi DataFrame thành DataFrame khác | `.transform(df)` | `StringIndexer model`, `OneHotEncoderModel`, `VectorAssembler` |
| **Estimator** | Học từ data và tạo ra Transformer | `.fit(df)` → trả về Transformer | `StringIndexer`, `StandardScaler`, `LogisticRegression` |

> ⚠️ **Lưu ý thi:** `Estimator.fit()` tạo ra một `Transformer`. Ví dụ: `StringIndexer` là Estimator, sau khi `.fit()` tạo ra `StringIndexerModel` (là Transformer). Khi sử dụng trong Pipeline: chỉ cần gọi `Pipeline.fit()` một lần, pipeline tự động fit từng Estimator và transform dữ liệu.

---

### 3.4 Xây dựng Training Pipeline

**MLlib Pipeline** là chuỗi các stage (Estimator hoặc Transformer) được thực hiện theo thứ tự:

```
VectorAssembler → StandardScaler → LogisticRegression
```

**Lợi ích của Pipeline:**
- Đóng gói toàn bộ workflow từ preprocessing đến training
- Tránh data leakage (fit scaler chỉ trên training data)
- Dễ reproduce và deploy
- Tích hợp với MLflow để log và version control

---

### 3.5 Hyperparameter Tuning — Điều chỉnh siêu tham số

#### Phương pháp tuning

| Phương pháp | Mô tả | Khi nào dùng |
|------------|-------|-------------|
| **Grid Search** | Thử **tất cả** kết hợp hyperparameter trong không gian tìm kiếm | Không gian nhỏ, cần đảm bảo tìm best config |
| **Random Search** | Thử ngẫu nhiên N kết hợp | Không gian lớn, budget giới hạn |
| **Bayesian Search** | Dùng kết quả trước để hướng dẫn tìm kiếm tiếp theo (Gaussian Process) | Hiệu quả hơn Grid/Random khi hàm mục tiêu đắt |

> ⚠️ **Lưu ý thi:** **Bayesian search** hiệu quả hơn Grid/Random vì "học" từ kết quả trước để ưu tiên vùng có tiềm năng, không thử ngẫu nhiên.

#### Hyperopt với `fmin`

**Hyperopt** là thư viện Python cho Bayesian hyperparameter optimization, tích hợp sâu với Databricks/MLflow.

Phép toán `fmin` trong Hyperopt:
- Nhận vào: hàm objective (cần minimize), search space, algorithm, max_evals
- Trả về: best hyperparameter configuration

> ⚠️ **Lưu ý thi:** `fmin` = **minimize** hàm mục tiêu. Nếu muốn maximize (ví dụ: accuracy), cần trả về **giá trị âm** trong hàm objective (ví dụ: `-accuracy`).

#### Song song hóa tuning trên một node

Trong Databricks, Hyperopt có thể chạy song song nhiều trial trên một cluster bằng cách dùng `SparkTrials`:

- `SparkTrials(parallelism=N)`: Chạy N trial song song trên Spark executors
- Tích hợp tự động với MLflow — mỗi trial được log như một MLflow run riêng biệt

---

### 3.6 Cross-Validation

**Cross-validation** là kỹ thuật đánh giá model bằng cách chia dữ liệu thành K fold, train trên K-1 fold và validate trên fold còn lại, lặp lại K lần.

**Lợi ích của Cross-validation so với train-validation split:**
- **Ổn định hơn**: Ước lượng hiệu năng chính xác hơn vì dùng toàn bộ dữ liệu cho cả training và validation
- **Ít bị ảnh hưởng bởi random split**: Không phụ thuộc vào cách chia dữ liệu một lần

**Hạn chế của Cross-validation:**
- **Tốn kém tính toán**: Train K lần thay vì 1 lần
- **Chậm với dataset lớn**

**Trong Spark MLlib:** Dùng `CrossValidator` kết hợp với `ParamGridBuilder`

> ⚠️ **Lưu ý thi:** **Số model được train khi kết hợp Grid Search với Cross-validation:**
> - Grid search có N hyperparameter combinations
> - K-fold cross-validation
> - **Tổng số model được train = N × K**
> - Ví dụ: 3 learning rates × 3 max depths = 9 combinations, K=5 → train 45 models

---

### 3.7 Metrics đánh giá

#### Metrics cho Classification

| Metric | Công thức / Ý nghĩa | Khi nào dùng |
|--------|---------------------|-------------|
| **Accuracy** | (TP + TN) / Total | Dữ liệu cân bằng |
| **Precision** | TP / (TP + FP) | Cần giảm False Positive (ví dụ: spam filter) |
| **Recall** | TP / (TP + FN) | Cần giảm False Negative (ví dụ: bệnh ung thư) |
| **F1-score** | 2 × (Precision × Recall) / (P + R) | Cân bằng Precision và Recall, dùng khi imbalanced |
| **Log Loss** | −(y·log(p) + (1−y)·log(1−p)) | Đánh giá xác suất dự đoán, tốt cho probabilistic output |
| **AUC-ROC** | Diện tích dưới ROC curve | Không phụ thuộc ngưỡng, tốt cho imbalanced |

#### Metrics cho Regression

| Metric | Công thức | Đặc điểm |
|--------|----------|----------|
| **MAE** (Mean Absolute Error) | Mean(\|y - ŷ\|) | Robust với outlier, dễ diễn giải |
| **MSE** (Mean Squared Error) | Mean((y - ŷ)²) | Phạt nặng outlier |
| **RMSE** (Root MSE) | √MSE | Cùng đơn vị với y, phạt nặng outlier |
| **R²** (R-squared) | 1 - SS_res/SS_tot | Tỉ lệ variance được giải thích (0 đến 1) |

> ⚠️ **Lưu ý thi:**
> - **F1 cho imbalanced classification**, Accuracy cho balanced
> - **RMSE phạt nặng outlier** (vì bình phương sai số); MAE thì không
> - **R² = 1** nghĩa là model hoàn hảo; R² có thể âm (model tệ hơn cả baseline mean)
> - Khi target đã log-transform → **phải exp() prediction trước khi tính MAE/RMSE**

---

### 3.8 Bias-Variance Tradeoff và độ phức tạp mô hình

**Bias (Độ chệch):** Sai số do model quá đơn giản — không nắm bắt được pattern trong data (underfitting).

**Variance:** Sai số do model quá phức tạp — học cả noise trong training data (overfitting).

| Vấn đề | Dấu hiệu | Giải pháp |
|--------|----------|----------|
| **High Bias (Underfitting)** | Training error cao, validation error cao | Tăng độ phức tạp model, thêm features |
| **High Variance (Overfitting)** | Training error thấp, validation error cao | Regularization, thêm data, giảm complexity, dropout |
| **Optimal** | Training ≈ Validation, cả hai thấp | — |

> ⚠️ **Lưu ý thi:** Tăng độ phức tạp model → giảm bias nhưng tăng variance. Regularization (L1/L2) giảm variance bằng cách penalize model complexity.

---

## Phần 4: Model Deployment

### 4.1 So sánh Batch, Real-time và Streaming Inference

> ⚠️ **Lưu ý thi:** Đây là câu hỏi thường xuyên xuất hiện. Phân biệt rõ 3 phương thức:

| Tiêu chí | Batch Inference | Real-time Inference | Streaming Inference |
|---------|----------------|--------------------|--------------------|
| **Kích hoạt** | Theo lịch (schedule) | Theo yêu cầu (on-demand) | Liên tục (continuous) |
| **Latency** | Cao (phút đến giờ) | Rất thấp (< 100ms) | Thấp (giây đến phút) |
| **Throughput** | Rất cao | Thấp đến vừa | Vừa đến cao |
| **Tool trên Databricks** | pandas / Spark DataFrame | Model Serving Endpoint | Delta Live Tables |
| **Use case** | Chấm điểm tín dụng hàng đêm, report tuần | Fraud detection, recommendation tức thì | Phân tích clickstream, IoT monitoring |

---

### 4.2 Triển khai custom model lên Model Serving Endpoint

**Quy trình triển khai:**
1. **Register model** vào Unity Catalog (hoặc Workspace Registry)
2. Vào **Serving** tab trong Databricks UI
3. Tạo endpoint, chọn model và version
4. Chọn compute size (workload type: CPU/GPU, size: Small/Medium/Large)
5. Deploy → endpoint tự động scale

**Custom model với MLflow PyFunc:**
- Dùng khi model không thuộc framework standard (scikit-learn, XGBoost, v.v.)
- Cần implement 2 method: `load_context()` (khởi tạo artifacts một lần) và `predict()` (logic inference)
- Log bằng `mlflow.pyfunc.log_model()`

> ⚠️ **Lưu ý thi:** Model Serving Endpoint là **serverless** — tự động scale lên/xuống theo traffic, không cần quản lý infrastructure.

---

### 4.3 Batch Inference với pandas

**Dùng pandas cho batch inference khi:**
- Dataset vừa đủ để load vào memory của driver/single node
- Model là single-node model (scikit-learn, XGBoost, v.v.)
- Cần xử lý trong pandas environment

**Quy trình:**
1. Load data vào pandas DataFrame
2. Load model từ MLflow: `mlflow.pyfunc.load_model(model_uri)`
3. Gọi `model.predict(pandas_df)`
4. Lưu kết quả

**Khi dataset lớn:** Dùng Spark DataFrame kết hợp `score_batch()` của Feature Engineering Client, hoặc UDF wrap model.

---

### 4.4 Streaming Inference với Delta Live Tables

**Delta Live Tables (DLT)** là framework declarative để xây dựng reliable data pipeline trên Databricks.

**Streaming inference với DLT:**
- Định nghĩa pipeline đọc data từ streaming source (Kafka, Delta table, v.v.)
- Apply model predictions như một transformation trong DLT pipeline
- DLT tự động xử lý checkpointing, retry, và error handling

**Ưu điểm:**
- Declarative (chỉ cần định nghĩa "cần gì", không phải "làm thế nào")
- Tích hợp sẵn với Delta Lake
- Tự động quản lý trạng thái streaming

> ⚠️ **Lưu ý thi:** Streaming inference trên Databricks dùng **Delta Live Tables** — đây là câu trả lời chuẩn khi đề hỏi về cách thực hiện streaming inference.

---

### 4.5 Triển khai và truy vấn model cho Real-time Inference

**Deploy:**
1. Model đã register trong Unity Catalog
2. Tạo serving endpoint qua UI hoặc REST API
3. Chờ endpoint ở trạng thái "Ready"

**Query endpoint:**
- REST API: `POST /serving-endpoints/{endpoint-name}/invocations`
- Request body: JSON với dữ liệu đầu vào
- Hỗ trợ: MLflow Deployments SDK, Python `requests`, curl

**Security:**
- Xác thực bằng Databricks API token
- AES-256 encryption at rest, TLS 1.2+ in transit

---

### 4.6 Traffic Splitting giữa các Endpoint

**Traffic splitting** cho phép phân chia traffic giữa nhiều model trên cùng một endpoint.

**Cấu hình qua `traffic_config`:**
- Ví dụ: 90% traffic → Model v1 (Champion), 10% traffic → Model v2 (Challenger)
- Điều chỉnh phân chia qua UI hoặc REST API mà không cần tạo lại endpoint

**Use cases:**
- **A/B testing**: So sánh hiệu năng 2 model với real traffic
- **Canary deployment**: Deploy dần với nhỏ traffic trước (1-10%)
- **Shadow testing**: Gửi bản sao traffic đến model mới (không ảnh hưởng user)

> ⚠️ **Lưu ý thi:**
> - Tất cả model trên cùng một endpoint phải có **cùng task type** (không thể mix classification và regression)
> - Để test riêng từng model (bỏ qua traffic config): query trực tiếp `POST /serving-endpoints/{endpoint}/served-models/{model-name}/invocations`
> - Traffic percentages phải cộng lại đúng 100%

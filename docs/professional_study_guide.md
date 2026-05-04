# Databricks Certified Machine Learning Professional — Tài liệu ôn thi

> Tài liệu này được tổng hợp từ tài liệu chính thức của Databricks, dịch sang tiếng Việt, giữ nguyên các thuật ngữ kỹ thuật tiếng Anh. Các mục được đánh dấu ⚠️ là những điểm quan trọng, thường xuất hiện trong đề thi.

---

## Thông tin kỳ thi

| Hạng mục | Chi tiết |
|----------|----------|
| Số câu hỏi | 59 câu trắc nghiệm |
| Thời gian | 120 phút |
| Lệ phí | 200 USD |
| Hiệu lực | 2 năm |
| Hình thức | Online proctored hoặc test center |

**Tỷ trọng nội dung:**

| Phần | Tỷ trọng |
|------|---------|
| Model Development | 44% |
| MLOps | 44% |
| Model Deployment | 12% |

---

## Phần 1: Model Development

### 1.1 Using Spark ML — Sử dụng SparkML

#### Khi nào nên dùng SparkML

> ⚠️ **Lưu ý thi:** Đây là câu hỏi quan trọng về kiến trúc. Hiểu rõ trade-off:

**Dùng SparkML (distributed) khi:**
- Dataset **quá lớn** để load vào memory của một node
- Model cần train trên **hàng tỷ record** hoặc dataset nhiều terabytes
- Cần tận dụng cluster distributed computing
- Pipeline đã có nhiều bước Spark transformation

**Dùng single-node model khi:**
- Dataset **vừa với memory** của một node (thường < vài chục GB)
- Model phức tạp (XGBoost, LightGBM, deep learning) vốn **chạy tốt hơn trên single node**
- Cần cross-validation hoặc hyperparameter tuning phức tạp

> ⚠️ **Lưu ý thi:** Databricks khuyến nghị: **"When possible, train neural networks on a single machine; distributed code is more complex and slower due to communication overhead."** Chỉ chuyển sang distributed khi model/data vượt quá memory single-node.

#### Xây dựng Pipeline bằng SparkML

**MLlib Pipeline** (`pyspark.ml.Pipeline`) là chuỗi các stage được thực hiện theo thứ tự. Mỗi stage là Estimator hoặc Transformer.

**Quy trình điển hình:**
```
StringIndexer → OneHotEncoder → VectorAssembler → StandardScaler → LogisticRegression
```

Sau khi `Pipeline.fit(training_data)` → trả về `PipelineModel` (Transformer), dùng để `.transform()` trên test data.

**Lợi ích:**
- Tự động fit Estimators và transform data theo thứ tự
- Tránh data leakage khi kết hợp với CrossValidator
- Deploy pipeline như một artifact duy nhất

#### Estimator vs Transformer trong SparkML

| Khái niệm | Phương thức | Ví dụ cụ thể |
|-----------|------------|-------------|
| **Estimator** | `.fit(df)` → trả về Transformer | `StringIndexer`, `StandardScaler`, `LogisticRegression`, `GBTClassifier` |
| **Transformer** | `.transform(df)` → trả về DataFrame | `StringIndexerModel`, `StandardScalerModel`, `PipelineModel`, `VectorAssembler` |

**Quy tắc ghi nhớ:**
- **Estimator học từ data** → tạo ra Transformer
- **Transformer áp dụng transformation** đã học lên data mới

#### Tinh chỉnh (Tuning) model SparkML bằng MLlib

**CrossValidator** với **ParamGridBuilder:**
- `ParamGridBuilder`: Định nghĩa grid các hyperparameter cần thử
- `CrossValidator`: Thực hiện K-fold cross-validation với mỗi config trong grid
- Kết hợp với Evaluator (ví dụ: `BinaryClassificationEvaluator`, `RegressionEvaluator`)

> ⚠️ **Lưu ý thi:** Số model được train = **N_combinations × K_folds**.

**TrainValidationSplit:** Alternative đơn giản hơn CrossValidator — chỉ split 1 lần (train/validation) thay vì K-fold.

#### Đánh giá model SparkML

**Classification Evaluators:**
- `BinaryClassificationEvaluator`: AUC-ROC (default), AUC-PR
- `MulticlassClassificationEvaluator`: Accuracy, F1, Precision, Recall

**Regression Evaluators:**
- `RegressionEvaluator`: RMSE (default), MSE, MAE, R²

#### Scoring SparkML model — Batch và Streaming

**Batch scoring:**
- `PipelineModel.transform(batch_dataframe)` — áp dụng model trên toàn bộ batch DataFrame
- Kết quả trả về DataFrame với cột `prediction` (và `probability` cho classification)

**Streaming scoring:**
- Dùng Spark Structured Streaming: `model.transform(streaming_dataframe)`
- SparkML PipelineModel tương thích trực tiếp với streaming DataFrame
- Kết quả ghi vào Delta table hoặc sink khác theo thời gian thực

> ⚠️ **Lưu ý thi:** Chọn **SparkML** cho batch/streaming scoring khi:
> - Dataset lớn, không fit vào memory 1 node
> - Cần native Spark streaming integration
>
> Chọn **single-node model** cho real-time inference khi:
> - Latency yêu cầu rất thấp (< 100ms)
> - Model phức tạp (deep learning) hiệu quả hơn trên GPU/single node

---

### 1.2 Scaling và Tuning — Mở rộng và Tối ưu hóa

#### Mở rộng training pipeline phân tán

**Pandas Function APIs / Pandas UDFs:**

Pandas UDFs (còn gọi là vectorized UDFs) dùng Apache Arrow để truyền dữ liệu hiệu quả giữa JVM và Python, nhanh hơn row-at-a-time UDFs lên đến **100 lần**.

**Các loại Pandas UDF:**

| Loại | Input / Output | Phù hợp với |
|------|---------------|------------|
| **Series to Series** | Series → Series (cùng length) | Vectorized transformation theo từng giá trị |
| **Iterator of Series** | Iterator[Series] → Iterator[Series] | Cần load state 1 lần (ví dụ: load ML model) |
| **Series to Scalar** | Series → scalar | Aggregation theo group |

> ⚠️ **Lưu ý thi:** **Iterator pattern** rất quan trọng cho ML inference: load model một lần trong iterator, apply cho nhiều batch → hiệu quả hơn nhiều so với load lại model cho mỗi row.

**Pandas Function APIs cho per-group training (applyInPandas):**
- `GroupedData.applyInPandas(func, schema)` — train một model riêng cho mỗi group
- Use case: Train model riêng cho từng store, từng region, từng user segment
- Kết quả là distributed training across groups, mỗi group chạy trên một executor riêng

#### Tinh chỉnh siêu tham số phân tán với Optuna + MLflow

**Optuna** là framework Python lightweight cho hyperparameter optimization với dynamic search space.

**Tích hợp MLflow 3.0 (Databricks Runtime 17.0 ML+):**
- `MlflowStorage`: Backend persistence sử dụng MLflow tracking server
  - `experiment_id`: Liên kết với MLflow experiment
  - `batch_flush_interval`: Thời gian giữa các batch flush (default: 1.0 giây)
  - `batch_size_threshold`: Số items tối đa trước khi flush (default: 100)
  - **Mục đích**: Tránh REST API throttling khi chạy nhiều trial song song
- `MlflowSparkStudy`: Phân phối trial execution trên Spark executors
  - Mỗi Spark worker chạy một số trial độc lập
  - Kết quả được tổng hợp về MLflow Tracking Server

**Sampler và Pruner mặc định:**
- `TPESampler` (Tree-structured Parzen Estimator): Bayesian-style — học từ kết quả trước
- `MedianPruner`: Dừng sớm các trial có kết quả trung gian kém hơn median

> ⚠️ **Lưu ý thi:** Optuna với `MlflowSparkStudy` thực hiện **distributed hyperparameter tuning** — mỗi trial chạy độc lập trên một Spark executor, kết quả tổng hợp qua MLflow. Cần configure `batch_flush_interval` và `batch_size_threshold` để tránh throttle API.

#### Tinh chỉnh siêu tham số phân tán với Ray

**Ray Tune** là module trong Ray ecosystem cho distributed hyperparameter search.

**Điểm mạnh của Ray:**
- Hỗ trợ nhiều search algorithm: Grid, Random, Bayesian (HyperOpt, Optuna), Population Based Training
- Tích hợp với MLflow để tracking
- Phù hợp với large-scale distributed training (deep learning)

**Ray kết nối với MLflow:**
```
Ray → MLflow Tracking → kết quả experiment, model artifacts
```

#### Vertical Scaling vs Horizontal Scaling cho ML workloads

> ⚠️ **Lưu ý thi:** Hiểu trade-off này:

| Tiêu chí | Vertical Scaling | Horizontal Scaling |
|---------|-----------------|-------------------|
| **Cách mở rộng** | Dùng node mạnh hơn (thêm CPU, RAM, GPU) | Thêm nhiều node vào cluster |
| **Phù hợp với** | Single-node models (XGBoost, deep learning trên GPU) | Data parallelism (SparkML, Spark pipeline) |
| **Độ phức tạp** | Thấp — code không thay đổi | Cao hơn — cần code hỗ trợ distributed |
| **Giới hạn** | Bị giới hạn bởi kích thước node tối đa | Gần như không giới hạn (theo lý thuyết) |
| **Cost** | Tốn kém với high-end node | Linh hoạt hơn với commodity nodes |

**Chiến lược chọn:**
- Thử vertical scaling trước (đơn giản hơn, ít overhead)
- Chuyển sang horizontal khi dataset/model vượt quá giới hạn memory/compute của single node

#### Model Parallelism vs Data Parallelism

| Loại | Mô tả | Khi nào dùng |
|------|-------|-------------|
| **Data Parallelism** | Chia data thành shard, mỗi node train trên shard riêng, đồng bộ gradient | Dataset lớn, model vừa fit vào memory 1 node |
| **Model Parallelism** | Chia model (layers/parameters) ra nhiều GPU/node | Model quá lớn để fit vào memory 1 GPU (ví dụ: LLM) |

**Trên Databricks:**
- **Data Parallelism**: TorchDistributor, DeepSpeed Distributor, Spark ML
- **Model Parallelism**: DeepSpeed (pipeline parallelism, ZeRO optimizer)

> ⚠️ **Lưu ý thi:** DeepSpeed hỗ trợ **cả hai** (model + data parallelism). TorchDistributor chủ yếu là data parallelism.

#### So sánh Ray và Spark cho distributed ML workloads

> ⚠️ **Lưu ý thi:** Câu hỏi quan trọng — Ray vs Spark:

| Tiêu chí | Ray | Apache Spark |
|---------|-----|-------------|
| **Chuyên môn** | **Logical parallelism** — ML training, tuning, serving | **Data parallelism** — ETL, analytics, data processing |
| **Phù hợp với** | Distributed deep learning, RL, hyperparameter tuning | SQL, DataFrames, batch processing lớn |
| **Thư viện ML** | Ray Train, Ray Tune, Ray Serve | MLlib, SparkML |
| **Tích hợp MLflow** | Có — kết nối MLflow tracking server | Có — tích hợp native |

**Kết hợp Ray và Spark:**
- Ray và Spark là **complementary** (bổ sung cho nhau), không phải thay thế
- Spark cho data preprocessing → Ray cho training → Ray Serve hoặc Model Serving cho deployment
- Databricks cho phép chạy Ray trên Spark cluster

#### Pandas Function API cho group-level training và inference

**Ứng dụng:**
- **Per-group training**: Train model riêng cho từng segment (store, region, sensor)
- **Per-group inference**: Áp dụng model tương ứng cho từng group

**Phương thức `applyInPandas`:**
- Nhận GroupedData → apply pandas function → trả về pandas DataFrame
- Spark tự động phân phối computation: mỗi group chạy trên một Spark task độc lập

**Ưu điểm:**
- Đơn giản hơn custom distributed frameworks
- Tận dụng Spark cluster cho nhiều group song song
- Phù hợp khi có hàng trăm/nghìn group nhỏ (multi-model training)

---

### 1.3 Advanced MLflow Usage — Sử dụng MLflow Nâng cao

#### Nested Runs trong MLflow

**Nested runs** cho phép tổ chức experiment tracking theo cấu trúc phân cấp (parent-child).

**Khi nào dùng nested runs:**
- Hyperparameter tuning: parent run = experiment, child runs = từng trial
- Cross-validation: parent run = overall CV, child runs = từng fold
- Multi-stage pipeline: parent run = pipeline, child runs = từng stage
- Ensemble training: parent run = ensemble, child runs = từng base model

**Cách tạo nested run:**
- `mlflow.start_run(nested=True)` bên trong một active run

**Lợi ích trong MLflow UI:**
- Xem tổng quan parent run và drill down vào child runs
- So sánh các child runs (từng trial/fold) trong context của parent

> ⚠️ **Lưu ý thi:** Nested runs giúp **theo dõi các thí nghiệm phức tạp** như hyperparameter search và cross-validation. Không dùng cho simple, single-step training.

#### Log Custom Metrics, Parameters và Artifacts bằng lập trình

**Custom metrics:**
- `mlflow.log_metric("custom_f2_score", 0.85, step=epoch)` — log metric với step (cho time series metrics)
- `mlflow.log_metrics(dict)` — log nhiều metrics cùng lúc

**Custom artifacts:**
- Bất kỳ file nào: `mlflow.log_artifact("path/to/file")`
- Thư mục: `mlflow.log_artifacts("path/to/dir", artifact_path="subdir")`
- Ví dụ: confusion matrix plot, feature importance chart, calibration curve, SHAP values

**Custom parameters:**
- `mlflow.log_param("feature_selection_method", "recursive")`

**Logging từ callback / custom framework:**
- Dùng `mlflow.get_tracking_uri()` để lấy tracking server URI
- Dùng `mlflow.active_run().info.run_id` để log vào run hiện tại từ bất kỳ đâu trong code

> ⚠️ **Lưu ý thi:** Log custom artifacts quan trọng vì: (1) lưu lại visualizations giúp audit sau này, (2) lưu preprocessing artifacts (scaler, encoder) cần thiết cho inference, (3) lưu feature importance để giải thích model.

#### Tạo Custom Model Objects với Real-time Feature Engineering

**MLflow PyFunc** (`mlflow.pyfunc`) cho phép tạo custom model class tích hợp feature engineering:

**Hai method bắt buộc:**
1. **`load_context(context)`**: Chạy một lần khi load model — dùng để load artifacts (model weights, scaler, feature list, v.v.)
2. **`predict(context, model_input)`**: Chạy mỗi lần có request — thực hiện feature engineering + inference

**Tích hợp real-time feature engineering:**
- Trong `predict()`: Gọi Feature Store API để lấy on-demand features
- Perform feature transformation trước khi inference
- Đảm bảo **training/serving consistency**: cùng logic feature engineering cho cả training và serving

> ⚠️ **Lưu ý thi:** Dùng PyFunc custom model khi cần: (1) kết hợp feature engineering + inference trong 1 artifact, (2) load external resources một lần (tiết kiệm latency), (3) xử lý business logic phức tạp trước/sau inference.

---

### 1.4 Advanced Feature Store Concepts — Khái niệm Feature Store Nâng cao

#### Point-in-time Correctness — Tính đúng đắn theo thời điểm

**Vấn đề:** Khi train model với time series data, nếu join feature không đúng thời điểm → **data leakage** (dữ liệu tương lai bị "rò rỉ" vào training).

**Ví dụ data leakage:**
- Observation timestamp: 2024-01-15
- Feature lookup không có point-in-time → có thể lấy feature value của ngày 2024-01-20
- Model vô tình "nhìn thấy" tương lai → kết quả tốt giả tạo khi training, thất bại trên production

**Databricks Feature Store giải quyết:**
- `FeatureEngineeringClient.create_training_set()` với `timestamp_lookup_key`
- Tự động join feature table theo **đúng thời điểm** của từng observation
- Đảm bảo feature value phản ánh trạng thái tại thời điểm observation, không phải thời điểm sau

> ⚠️ **Lưu ý thi:** Point-in-time correctness ngăn chặn **temporal data leakage**. Đây là yêu cầu bắt buộc khi làm việc với time series feature và fraud detection. Dùng `timestamp_keys` khi tạo feature table để hỗ trợ point-in-time lookup.

#### Xây dựng pipeline tự động với FeatureEngineering Client

**`FeatureEngineeringClient`** (Python API) là API hiện đại để tương tác với Feature Store trong Unity Catalog:

**Các thao tác chính:**
- `create_table()`: Tạo feature table mới trong Unity Catalog
- `write_table()`: Ghi/cập nhật dữ liệu vào feature table (batch hoặc streaming)
- `create_training_set()`: Tạo training dataset bằng cách join feature lookups
- `log_model()`: Log model kèm feature lookup specs vào MLflow
- `score_batch()`: Thực hiện batch inference với automatic feature lookup

**Automated feature computation pipeline:**
- Sử dụng Databricks Jobs/Workflows để schedule việc tính toán feature theo chu kỳ
- Pipeline: Raw data → Feature computation → `write_table()` → Feature table
- Model inference tự động lấy feature mới nhất từ feature table

#### Cấu hình Online Tables cho ứng dụng low-latency

**Online tables** là bản sao của offline feature table được tối ưu cho read nhanh (millisecond latency).

**Tạo online table bằng Databricks SDK:**
- Chỉ định nguồn (Delta table hoặc offline feature table)
- Sync mode: triggered (on-demand) hoặc continuous (near real-time)
- Primary key cho lookup

**Use cases:**
- Real-time fraud detection: lookup user features theo user_id
- Personalization: lookup user preferences tại thời điểm request
- Dynamic pricing: lookup context features cho mỗi transaction

> ⚠️ **Lưu ý thi:** Online tables cung cấp **millisecond latency** cho feature lookup. Offline feature tables có latency cao hơn (giây đến phút). Cần online table khi serving endpoint cần đọc feature tại request time.

#### Thiết kế giải pháp xử lý streaming data để tạo real-time features

**Kiến trúc streaming feature pipeline:**
1. **Ingest**: Kafka / Event Hubs / Kinesis → Databricks Auto Loader hoặc Spark Structured Streaming
2. **Transform**: Compute features (aggregation, normalization) trên streaming data
3. **Write**: `FeatureEngineeringClient.write_table()` với streaming DataFrame → cập nhật feature table liên tục
4. **Serve**: Online table sync từ offline feature table → sẵn sàng cho real-time lookup

**Lợi ích:**
- Feature luôn up-to-date với data mới nhất
- Giảm lag giữa event xảy ra và feature sẵn sàng cho inference

#### On-demand Features với Feature Serving

**On-demand features** là features được tính toán tại thời điểm inference (không được lưu trữ trước).

**Khi nào dùng on-demand features:**
- Feature phụ thuộc vào request context (input của request hiện tại)
- Feature không thể tính trước (ví dụ: khoảng cách từ user location đến shop)
- Feature cần tính từ raw input (text embedding, image features)

**Training/serving consistency với on-demand features:**
- Cùng logic tính feature được dùng khi training (qua `create_training_set`) và khi serving (qua feature serving endpoint)
- Tránh **training/serving skew** — sự khác biệt giữa feature được tính khi training và khi serving

> ⚠️ **Lưu ý thi:** On-demand features đảm bảo **consistent feature computation** giữa training và production. Khác với pre-computed features (tính trước, lưu vào bảng): on-demand features được tính tại thời điểm request.

---

## Phần 2: MLOps

### 2.1 Model Lifecycle Management — Quản lý Vòng đời Mô hình

#### Kiến trúc pipeline vòng đời mô hình

**Ba môi trường chuẩn:**

| Môi trường | Mục đích | Ai quản lý |
|-----------|---------|-----------|
| **Development** | Experiment, develop features và models | Data Scientists |
| **Staging** | Test code readiness, validation | Data Scientists + ML Engineers |
| **Production** | Deploy và operate ML pipeline | ML Engineers |

**Luồng promotion chuẩn (code promotion):**
```
Feature branch → Pull Request → Unit Tests (CI) → Integration Tests (CI) 
→ Merge to main → Release branch → Production deployment
```

**Các thành phần kiến trúc:**
- **Unity Catalog**: Riêng biệt cho dev, staging, prod — quản lý models, features, data
- **MLflow Tracking**: Server riêng cho từng môi trường
- **Databricks Workflows**: Orchestrate multi-stage pipeline (feature engineering → training → evaluation → deployment)
- **Git**: Version control cho tất cả code
- **Databricks Asset Bundles (DABs)**: Infrastructure-as-code để deploy ML assets

#### Ánh xạ Databricks features vào MLOps workflow

| Hoạt động MLOps | Databricks Feature |
|----------------|-------------------|
| Experiment tracking | MLflow Experiments và Runs |
| Model versioning | Unity Catalog Model Registry |
| Feature management | Feature Engineering (Unity Catalog) |
| Pipeline orchestration | Databricks Workflows / Jobs |
| Code deployment | Databricks Asset Bundles (DABs) |
| Model monitoring | Lakehouse Monitoring |
| Model serving | Mosaic AI Model Serving |
| CI/CD integration | GitHub Actions / Azure DevOps + DABs |
| Data versioning | Delta Lake + Delta tables |

---

### 2.2 Validation Testing — Kiểm thử và Xác minh

#### Unit Tests cho hàm trong Databricks Notebooks

**Unit test** kiểm tra từng hàm riêng lẻ với các đầu vào cụ thể và xác minh đầu ra mong đợi.

**Trong Databricks notebooks:**
- Dùng `pytest` hoặc `unittest` tích hợp trong notebook
- Tổ chức test trong cell riêng biệt hoặc module Python riêng
- Chạy: `%run ./test_module` hoặc sử dụng `dbutils.notebook.run()`

**Best practices:**
- Test hàm feature engineering độc lập (tách business logic khỏi Spark code)
- Test transformation functions (normalization, encoding, v.v.)
- Dùng small, representative fixtures — không cần Spark cluster cho unit test

> ⚠️ **Lưu ý thi:** Unit tests kiểm tra **từng hàm riêng lẻ** với input/output cụ thể. Không phụ thuộc vào external systems (không cần cluster, database). Nên chạy trong CI (trước khi merge code).

#### Các loại kiểm thử theo giai đoạn môi trường

| Loại test | Môi trường | Mục đích |
|-----------|-----------|---------|
| **Unit tests** | Dev + Staging (CI) | Kiểm tra từng hàm/component riêng lẻ |
| **Integration tests** | Staging (CI) | Kiểm tra toàn bộ pipeline (feature → train → evaluate → deploy) |
| **Smoke tests** | Production | Xác nhận endpoint healthy sau deployment |
| **Performance tests** | Staging | Kiểm tra latency, throughput trước khi đưa vào prod |

> ⚠️ **Lưu ý thi:** **Unit tests chạy trong Dev và Staging** (fast, no infra needed). **Integration tests chạy trong Staging** (cần đầy đủ infrastructure). **Không chạy integration tests trong Dev** (tốn kém và chậm).

#### Integration Tests cho hệ thống ML

**Integration test** kiểm tra toàn bộ hệ thống end-to-end với tất cả components tích hợp với nhau.

**Các pipeline cần integration test:**
1. **Feature engineering pipeline**: Raw data → Feature tables (kiểm tra schema, data types, null values)
2. **Training pipeline**: Features → Trained model (kiểm tra model được train và log thành công)
3. **Evaluation pipeline**: Model + test data → Metrics (kiểm tra metrics vượt ngưỡng tối thiểu)
4. **Deployment pipeline**: Model → Serving endpoint (kiểm tra endpoint health và response format)
5. **Inference pipeline**: Request → Prediction (kiểm tra end-to-end latency và correctness)

**Thách thức của integration tests cho ML:**
- Tốn thời gian và tài nguyên (cần cluster, data, model)
- Data và model thay đổi → test có thể bị stale
- Cần mock một số external dependencies để ổn định

#### So sánh cách tổ chức hàm và unit tests

**Centralized functions + tests (functions trong module riêng):**
- **Lợi ích**: Dễ test, reuse code, dễ maintain
- **Thách thức**: Cần refactor notebook code ra module

**Inline functions trong notebook:**
- **Lợi ích**: Đơn giản, ít boilerplate
- **Thách thức**: Khó test tự động, khó reuse

> ⚠️ **Lưu ý thi:** Best practice là tách business logic ra Python modules (`.py` files) và notebook chỉ là orchestration. Modules dễ test bằng `pytest` hơn là inline notebook code.

---

### 2.3 Environment Architectures — Kiến trúc Môi trường

#### Thiết kế và triển khai môi trường Databricks scalable

**Best practices cho ML environments:**

1. **Separation of concerns**: Dev, staging, prod là các Unity Catalog catalogs riêng biệt
2. **Least privilege access**: Dev có thể đọc prod data (read-only), không có write
3. **Dedicated clusters per environment**: Tránh chia sẻ cluster giữa dev và prod
4. **Separate MLflow tracking**: Mỗi environment có tracking server riêng để tránh lẫn lộn experiments

**Cấu trúc Unity Catalog theo environment:**
```
dev_catalog/
  ml_schema/
    feature_tables/
    models/
    
staging_catalog/
  ml_schema/
    feature_tables/
    models/

prod_catalog/
  ml_schema/
    feature_tables/
    models/
```

#### Databricks Asset Bundles (DABs) — Infrastructure as Code

**DABs** là công cụ giúp áp dụng software engineering best practices (source control, code review, testing, CI/CD) cho data và AI projects.

**Những gì DABs quản lý:**
- Cloud infrastructure và workspace configurations
- Source files (notebooks, Python scripts)
- **Databricks resources**: Jobs, pipelines, dashboards, **Model Serving endpoints**
- Unit và integration tests

**Cách DABs hoạt động:**
- Metadata được định nghĩa trong **YAML files**
- Databricks CLI validate, deploy, execute bundles dựa trên YAML
- Hỗ trợ nhiều **targets** (dev, staging, prod) trong cùng bundle

**Điều kiện sử dụng:**
- Databricks CLI version v0.218.0 trở lên
- Workspace với workspace files enabled (default trong Runtime 11.3 LTS+)

**Khởi tạo bundle:**
```
databricks bundle init
databricks bundle validate
databricks bundle deploy -t <target>
databricks bundle run -t <target> <job-name>
```

#### Cấu hình ML assets bằng DABs

**Định nghĩa MLflow Experiment trong YAML:**
- Correspond với request payload của API tạo experiment
- Khai báo experiment name, artifact location, tags

**Định nghĩa Job/Pipeline trong YAML:**
- Correspond với create job API
- Cấu hình cluster settings, task dependencies, schedule

**Định nghĩa Registered Model (Unity Catalog) trong YAML:**
- Correspond với create Unity Catalog model API
- Khai báo catalog, schema, model name, permissions

**Định nghĩa Model Serving Endpoint trong YAML:**
- Khai báo endpoint name, served models, traffic config, scale-to-zero

> ⚠️ **Lưu ý thi:** DABs = **Databricks Asset Bundles** — dùng YAML để khai báo ML assets (experiments, models, endpoints, jobs) và deploy bằng CLI. Đây là cách Databricks khuyến nghị để quản lý ML infrastructure as code.

**MLOps Stacks:**
- Template DABs sẵn có theo production best practices
- Khởi tạo: `databricks bundle init mlops-stacks`
- Ba loại project: CICD_and_Project, Project_Only, CICD_Only

---

### 2.4 Automated Retraining — Tự động Huấn luyện lại

#### Triển khai quy trình retraining tự động

**Khi nào cần retraining:**
- **Data drift**: Phân phối của input data thay đổi so với lúc training
- **Model performance degradation**: Metric giảm dưới ngưỡng định sẵn (ví dụ: accuracy < 0.85)
- **Concept drift**: Quan hệ giữa features và target thay đổi theo thời gian
- **Scheduled retraining**: Định kỳ (hàng tuần, hàng tháng) bất kể drift

**Kiến trúc retraining pipeline:**
1. **Trigger**: Alert từ Lakehouse Monitoring (drift detected / performance degradation) hoặc schedule
2. **Data preparation**: Lấy data mới nhất từ feature store / raw tables
3. **Training**: Chạy training pipeline với data mới
4. **Evaluation**: So sánh model mới với Champion model hiện tại
5. **Conditional deployment**: Chỉ deploy nếu model mới tốt hơn

**Trên Databricks:** Dùng **Databricks Jobs** để orchestrate toàn bộ pipeline, có thể trigger bởi alert hoặc schedule.

> ⚠️ **Lưu ý thi:** Retraining pipeline cần có **evaluation gate** — không tự động deploy model mới mà phải so sánh với Champion. Chỉ promote Challenger thành Champion khi Challenger có metric tốt hơn.

#### Chiến lược chọn model tốt nhất trong retraining

**Champion-Challenger pattern:**
- **Champion**: Model đang chạy production (alias `@Champion`)
- **Challenger**: Model mới được train (alias `@Challenger`)
- Sau retraining: so sánh Champion vs Challenger trên **holdout set hoặc production data**
- Promote Challenger thành Champion nếu: metric tốt hơn VÀ vượt ngưỡng tối thiểu

**Tiêu chí chọn model:**
- Primary metric (ví dụ: AUC-ROC cho classification)
- Secondary metrics (Precision, Recall tùy business requirement)
- Inference latency (không được tăng quá ngưỡng định sẵn)
- Model complexity / interpretability

**Automation trong MLflow:**
- Dùng `MlflowClient.search_runs()` để so sánh model mới vs cũ
- `MlflowClient.set_registered_model_alias()` để promote model thắng

---

### 2.5 Drift Detection và Lakehouse Monitoring

#### Các loại bảng trong Lakehouse Monitoring

> ⚠️ **Lưu ý thi:** Phân biệt rõ 3 loại profile:

| Loại | Dùng cho | Đặc điểm |
|------|---------|----------|
| **Snapshot table** | Tables không có timestamp | Tính metrics trên toàn bộ dataset, không có time dimension |
| **Time series table** | Tables có timestamp column | Tính metrics theo time windows, theo dõi xu hướng theo thời gian |
| **Inference table** | Model request logs (input + prediction + optional label) | Theo dõi model performance, data drift, prediction distribution |

**Khi nào dùng loại nào:**
- **Snapshot**: Kiểm tra data quality định kỳ (không có natural time dimension)
- **Time series**: Monitoring feature drift theo thời gian
- **Inference**: Monitoring model performance và data quality cho serving endpoint — quan trọng nhất cho MLOps

#### Xây dựng monitor cho từng loại bảng

**Tạo monitor qua UI hoặc API:**
- Chỉ định: loại profile (snapshot/time series/inference), primary table, baseline table (optional)
- Databricks tự động tạo:
  - **Profile metrics table**: Summary statistics của data
  - **Drift metrics table**: Statistical tests so sánh với baseline hoặc giữa các time windows
  - **Dashboard**: Visualization tự động

**Baseline table:**
- Dataset "reference" — phản ánh expected distribution của data
- Thường là: training data hoặc data từ thời điểm model deployment
- Drift được đo bằng cách so sánh current window với baseline

#### Statistical Tests để phát hiện Drift

> ⚠️ **Lưu ý thi:** Biết test nào dùng cho loại data nào:

| Loại dữ liệu | Statistical Test | Mô tả |
|-------------|----------------|-------|
| **Dữ liệu liên tục** (numerical) | Wasserstein distance, KS-test (Kolmogorov-Smirnov) | Đo sự khác biệt giữa hai phân phối liên tục |
| **Dữ liệu phân loại** (categorical) | Chi-square test, Jensen-Shannon divergence | Đo sự khác biệt giữa hai phân phối rời rạc |
| **Model predictions** | PSI (Population Stability Index) | Đo sự dịch chuyển trong phân phối prediction |

**Diễn giải kết quả:**
- P-value thấp → drift có ý nghĩa thống kê
- Nhưng drift thống kê không nhất thiết là drift quan trọng về business — cần kết hợp với model performance metrics

#### Thiết kế cơ chế cảnh báo (Alerting)

**Alerting dựa trên metric tables:**
- Databricks Lakehouse Monitoring tạo metric tables (Delta tables)
- Tạo **Databricks SQL Alert** hoặc **Databricks Notification** dựa trên query metric table
- Trigger alert khi metric vượt ngưỡng định sẵn

**Ví dụ alert conditions:**
- `drift_score > 0.2` (Wasserstein distance vượt ngưỡng)
- `p_value < 0.05` (chi-square test có ý nghĩa)
- `accuracy < 0.80` (model performance giảm)
- `null_fraction > 0.1` (data quality issue)

**Notification channels:**
- Email, Slack, PagerDuty (qua Databricks notification system)
- Có thể trigger Databricks Job (ví dụ: tự động retraining pipeline)

> ⚠️ **Lưu ý thi:** Alert được cấu hình trên **metric tables** (Delta tables) sinh ra bởi Lakehouse Monitoring. Không phải alert trực tiếp trên raw data.

#### Phát hiện Data Drift

**Phương pháp 1: So sánh với baseline**
- Baseline = training data hoặc data từ lúc deploy
- Tính statistical test giữa current window và baseline
- Phát hiện khi input distribution thay đổi so với lúc model được train

**Phương pháp 2: So sánh giữa các time windows liên tiếp**
- So sánh window hiện tại (ví dụ: tuần này) với window trước (tuần trước)
- Phát hiện trend và đột biến trong data

#### Đánh giá xu hướng hiệu năng model theo thời gian

**Inference table monitoring:**
- Log model inputs, predictions, và ground truth labels (nếu có) vào inference table
- Lakehouse Monitoring tính metrics theo time windows: accuracy, F1, RMSE, v.v.
- Visualize trend trong dashboard

**Khi không có immediate ground truth:**
- Monitor **proxy metrics**: prediction distribution shift, confidence score distribution
- Khi labels có được (delayed feedback): backfill inference table và trigger evaluation

#### Custom Metrics trong Lakehouse Monitoring

**Định nghĩa custom metrics:**
- Viết hàm Python/SQL tính metric tùy chỉnh
- Register vào monitor configuration
- Custom metrics được tính vào profile metrics table

**Ví dụ custom metrics:**
- Business-specific KPI (ví dụ: tỉ lệ conversion từ model predictions)
- Fairness metrics (ví dụ: demographic parity ratio)
- Domain-specific quality metrics

#### Feature Slicing và Data Granularity

**Feature slicing** cho phép tính metrics cho **subsets** (slice) của data:
- Ví dụ: xem drift riêng cho từng category, region, user segment
- Phát hiện drift chỉ ảnh hưởng một phần dữ liệu (không thấy khi xem toàn bộ)

**Mức độ granularity khác nhau:**
- Global: Tính trên toàn bộ data
- Per-feature: Tính cho từng feature riêng biệt
- Per-slice: Tính cho subset theo điều kiện cụ thể

> ⚠️ **Lưu ý thi:** Feature slicing quan trọng để phát hiện **hidden drift** trong subgroups — drift tổng thể nhỏ nhưng một nhóm dữ liệu cụ thể có drift lớn.

#### Monitoring Endpoint Health

**Infrastructure metrics** cần theo dõi cho serving endpoint:

| Metric | Ý nghĩa | Ngưỡng alert điển hình |
|--------|---------|----------------------|
| **Latency** (p50, p95, p99) | Thời gian response | > 200ms cho real-time |
| **Request rate** | Số requests/giây | Bất thường so với baseline |
| **Error rate** | % requests bị lỗi | > 1-5% |
| **CPU/Memory usage** | Resource utilization | > 80% |
| **Token per second** | Với LLM endpoints | Dưới SLA |

> ⚠️ **Lưu ý thi:** Endpoint health monitoring là **infrastructure-level**, khác với model performance monitoring. Cả hai đều cần thiết cho production MLOps.

---

## Phần 3: Model Deployment

### 3.1 Deployment Strategies — Chiến lược Triển khai

#### So sánh Blue-Green và Canary

> ⚠️ **Lưu ý thi:** Đây là câu hỏi thường xuyên — phân biệt rõ hai chiến lược:

| Tiêu chí | Blue-Green | Canary |
|---------|-----------|--------|
| **Cơ chế** | Hai endpoint đầy đủ: Blue (current) và Green (new). Switch traffic **toàn bộ** | Dần dần tăng traffic từ 0% → 10% → 50% → 100% cho version mới |
| **Rollback** | **Rất nhanh**: Switch traffic về Blue ngay lập tức | Chậm hơn: Giảm traffic về 0% cho version mới |
| **Chi phí** | **Gấp đôi infrastructure** (chạy 2 endpoint cùng lúc) | **Tiết kiệm** hơn (chỉ một endpoint với traffic split) |
| **Risk** | Vẫn có risk khi switch toàn bộ traffic | **Ít risk hơn** (phát hiện vấn đề khi còn ít traffic) |
| **Tốt nhất cho** | Zero-downtime cutover, rollback đơn giản | Validate với real traffic, tránh blast radius lớn |
| **Khi nào dùng** | Strict uptime requirement, cần rollback ngay lập tức | High-traffic ứng dụng, cần validate với real users trước |

**Deployment strategies khác:**
- **Shadow testing**: Copy real traffic đến model mới, so sánh kết quả không ảnh hưởng users
- **A/B testing**: Split traffic đều 50/50 để so sánh 2 model variants

#### Triển khai chiến lược rollout bằng Databricks Model Serving

**Trên Databricks Model Serving:**
- Mỗi endpoint hỗ trợ nhiều `served_models` với `traffic_percentage`
- Canary: Thêm model version mới với traffic_percentage nhỏ (5-10%), tăng dần
- Blue-green: Duy trì 2 endpoint riêng biệt, dùng reverse proxy hoặc DNS để switch

**Cập nhật traffic config (không cần tạo lại endpoint):**
- `PUT /serving-endpoints/{name}/config` với `traffic_config` mới
- Thay đổi có hiệu lực sau vài giây đến vài phút

> ⚠️ **Lưu ý thi:**
> - **Canary phù hợp cho high-traffic** để minimize blast radius
> - **Blue-green cho strict zero-downtime** — rollback là single switch, không có downtime
> - Trong Databricks: traffic split = **canary**; 2 endpoint riêng + switch = **blue-green**

---

### 3.2 Custom Model Serving — Triển khai Model Tùy chỉnh

#### Đăng ký Custom PyFunc Model và Log Custom Artifacts trong Unity Catalog

**Custom PyFunc model:**
- Class kế thừa `mlflow.pyfunc.PythonModel`
- Method bắt buộc: `load_context(context)` và `predict(context, model_input)`
- Log bằng: `mlflow.pyfunc.log_model(artifact_path, python_model=MyModel(), artifacts={...})`

**Log custom artifacts:**
- Trong `mlflow.pyfunc.log_model()`: truyền `artifacts` dict
- Key = artifact name, Value = local path của artifact file
- Artifacts được download về `context.artifacts["artifact_name"]` khi `load_context` chạy

**Register trong Unity Catalog:**
- `mlflow.register_model(model_uri, "catalog.schema.model_name")`
- Cần model signature (input/output schema)
- Cần `CREATE MODEL` privilege trên schema

#### Truy vấn Custom Models qua REST API hoặc MLflow Deployments SDK

**REST API:**
- Endpoint URL: `https://<workspace>/serving-endpoints/<endpoint-name>/invocations`
- Method: `POST`
- Header: `Authorization: Bearer <token>`
- Body: JSON với format tương ứng model input

**MLflow Deployments SDK:**
- `mlflow.deployments.get_deploy_client(target_uri)`
- `client.predict(endpoint="endpoint-name", inputs={...})`
- Cùng API cho cả Databricks-hosted và external models

> ⚠️ **Lưu ý thi:** MLflow Deployments SDK cung cấp **unified interface** — cùng code có thể query cả custom models lẫn foundation models (OpenAI, Anthropic, v.v.) mà không cần thay đổi code.

#### Triển khai Custom Model Objects

**3 cách triển khai:**

| Cách | Khi nào dùng |
|------|-------------|
| **MLflow Deployments SDK** | Programmatic deployment, CI/CD pipeline |
| **REST API** | Bất kỳ ngôn ngữ nào, fine-grained control |
| **Databricks UI** | Manual deployment, không cần code |

**Quy trình đầy đủ:**
1. Viết `PythonModel` class (PyFunc)
2. Log model với `mlflow.pyfunc.log_model()` kèm artifacts
3. Register model vào Unity Catalog
4. Tạo/cập nhật serving endpoint với model version mới
5. Test endpoint bằng REST API hoặc SDK
6. Monitor endpoint health và model performance

> ⚠️ **Lưu ý thi:** Custom model (`mlflow.pyfunc`) là lựa chọn khi model không thuộc các framework standard (scikit-learn, PyTorch, v.v.) hoặc cần custom preprocessing/postprocessing logic được đóng gói cùng model artifact.

# Notes ôn thi Databricks Certified Machine Learning Professional

## 1. Bài thi đang ôn
Tỷ trọng nội dung hiện tại:

- `Model Development`: `44%`
- `MLOps`: `44%`
- `Model Deployment`: `12%`

Hai nguồn chính dùng để xác định đề cương ôn tập:

- Certification page: https://www.databricks.com/learn/certification/machine-learning-professional
- Exam guide hiện hành: https://www.databricks.com/sites/default/files/2025-10/databricks-certified-machine-learning-professional-exam-guide-september.pdf

## 2. Nhóm tài liệu

4 nhóm tài liệu ôn tập chính

1. `Databricks Academy`
2. `Databricks documentation`
3. `Udemy practice tests`
4. `Bộ đề ngoài`


## 3. Bộ course Databricks nên học

Tập trung ngay vào 2 course này:

- `Machine Learning at Scale`
- `Advanced Machine Learning Operations`

Đây là 2 module Databricks đang dùng trong `Professional ML Practitioners` pathway và cũng là 2 course được exam guide hiện hành nhắc tới.

## 4. Bộ docs nên đọc để khóa kiến thức

Không cần đọc toàn bộ docs. Chỉ cần đọc những mảng bám sát blueprint.

Ghi chú:

- Databricks docs được tổ chức theo từng cloud
- Doc ở dưới đang là link của AWS

### 4.1. Tổng quan ML trên Databricks

- https://docs.databricks.com/en/machine-learning/index.html

Nên đọc mục này trước để nhìn ra hết các khối lớn: training, feature engineering, model serving, monitoring, MLflow.

### 4.2. MLflow

- https://docs.databricks.com/en/mlflow/index.html

Đây là phần cần ưu tiên rất cao vì nó ăn vào cả `Model Development` lẫn `MLOps`.

Những gì cần học ở đây:

- experiment tracking
- log params, metrics, artifacts
- model packaging
- registered model
- versioning và lifecycle

### 4.3. Feature Store / Feature Engineering

- https://docs.databricks.com/en/machine-learning/feature-store/index.html

Phần này giúp tránh mơ hồ về:

- feature table
- quản lý feature dùng chung
- training và inference consistency
- reuse feature theo production flow

### 4.4. Model Serving

- https://docs.databricks.com/en/machine-learning/model-serving/index.html

Phần này cần đọc để chắc các câu hỏi về:

- deploy endpoint
- rollout version
- serving flow
- inference logging
- khi nào dùng serving thay vì batch inference

### 4.5. Lakehouse Monitoring

- https://docs.databricks.com/en/lakehouse-monitoring/index.html

Đây là phần rất quan trọng cho `MLOps`.

Cần học:

- monitor là gì
- inference profile là gì
- baseline table là gì
- drift được theo dõi thế nào
- metric tables và dashboard dùng ra sao

### 4.6. CI/CD và MLOps

- https://docs.databricks.com/en/machine-learning/mlops/ci-cd-for-ml.html
- https://docs.databricks.com/en/dev-tools/ci-cd/
- https://docs.databricks.com/en/dev-tools/bundles
- https://docs.databricks.com/en/dev-tools/bundles/mlops-stacks

Phần này cần đọc để chắc các câu hỏi về:

- CI/CD cho ML trên Databricks
- test strategy
- dev/staging/prod separation
- bundles
- MLOps stacks

### 4.7. Jobs / Workflows

- https://docs.databricks.com/en/jobs/

Phần này dùng để ôn:

- orchestration
- retraining workflows
- scheduling
- multi-task jobs

### 4.8. Databricks Runtime for Machine Learning

- https://docs.databricks.com/en/machine-learning/databricks-runtime-ml.html

Phần này dùng để ôn:

- runtime ML
- thư viện built-in
- lý do dùng runtime ML trong workflow training

## 5. Tài liệu ngoài nên dùng để luyện đề

### 5.1. Nguồn được nhắc nhiều trong review Reddit

#### 1. Skillcertpro

- Link:
  - https://skillcertpro.com/product/databricks-machine-learning-professional-exam-questions/
- Giá khi kiểm tra ngày `2026-04-30`:
  - `19.99` USD
- Snapshot:
  - `380` câu hỏi
  - `7` mock exams

#### 2. Certsafari

- Link:
  - https://www.certsafari.com/databricks/ml-engineer-professional
- Giá:
  - `Free`
- Snapshot:
  - `330` exam-style practice questions

### 5.2. Khóa học hỗ trợ trên Udemy


#### 1. Databricks Certified Machine Learning Associate Exam Guide

- Link:
  - https://www.udemy.com/course/databricks-machine-learning/
- Snapshot:
  - rating khoảng `3.8/5`
  - khoảng `6,785 students`
  - `Last updated 11/2025`
  - `4h 39m` video

#### 2. MLflow for MLOps & LLMOps: Master MLflow with Databricks

- Link:
  - https://www.udemy.com/course/mlflow-for-mlops-llmops-master-mlflow-with-databricks/
- Snapshot:
  - `6h 28m` video
  - có các phần về `MLflow tracking`, `Model Registry`, `serving`, và tích hợp với `Databricks`

### 5.3. Practice exams trên Udemy

Udemy chỉ nên được xem như nguồn `mock exam`, không nên xem là nguồn chân lý kỹ thuật.

#### 1. Databricks Machine Learning Professional: Practice Exam 2026

- Link:
  - https://www.udemy.com/course/databricks-machine-learning-professional-practice-test/
- Snapshot:
  - rating khoảng `4.0/5`
  - khoảng `1,484 students`
  - `Last updated 4/2026`
  - khoảng `145` câu

#### 2. Databricks Certified Machine Learning Professional - Exams

- Link:
  - https://www.udemy.com/course/databricks-certified-machine-learning-professional-exams/
- Snapshot:
  - rating khoảng `3.9/5`
  - khoảng `4,316 students`
  - `Last updated 1/2026`
  - `6` mock exams
  - khoảng `300+` câu

#### 3. Databricks Machine Learning Professional Exam Tests 2026

- Link:
  - https://www.udemy.com/course/databricks-machine-learning-professional-exam-tests-2025/
- Giá:
  - `Dynamic on Udemy`
- Snapshot:
  - rating khoảng `4.6/5`
  - `Highest Rated`
  - `Bestseller`
  - `Last updated 1/2026`
  - khoảng `800+` câu


#### 4. Certification Databricks Machine Learning Professional

- Link:
  - https://www.udemy.com/course/certification-databricks-machine-learning-professional/
- Snapshot:
  - rating khoảng `4.5/5`
  - khoảng `494 students`
  - `Last updated 10/2025`
  - `341` câu
- Lý do nên xem:
  - cách trình bày bám domain
  - hợp để luyện theo blueprint thay vì luyện ngẫu nhiên

## 6. Cách kiếm voucher thi Databricks

Theo Knowledge Base chính thức của Databricks, voucher thường không phải thứ có thể xin đại trà. Các nguồn voucher chủ yếu là:

- `Databricks events`
- `beta exams`
- `partner organizations`
- `pre-purchased credits`

Nguồn xác nhận:

- Voucher KB:
  - https://kb.databricks.com/en_US/training/how-do-i-request-a-certification-voucher

### 6.1. Cách kiếm: Learning Festival

Trong đợt `Databricks Learning Festival (Self-paced): Global` từ `16 March 2026` đến `03 April 2026`, Databricks công bố:

- hoàn thành ít nhất `1` learning pathway hợp lệ trong thời gian event
- sẽ nhận `50% discount` cho bất kỳ Databricks Certification nào
- và `20% discount` cho Databricks Academy Labs

Lưu ý trong đợt đó:
- voucher thường có hiệu lực khoảng `90 ngày`
- với đợt này, voucher được phân phối ngày `09 April 2026`
- voucher hết hạn trước `08 July 2026`
- voucher không cộng dồn với offer khác hoặc success credits

Nguồn:

- Event:
  - https://community.databricks.com/t5/learning-events/databricks-learning-festival-self-paced-global/ev-p/150223
- FAQ:
  - https://community.databricks.com/t5/training-offerings/faq-for-virtual-learning-festival-16-march-03-april-2026/td-p/150220


### 6.2. Cách theo dõi voucher

Nên canh ở:

- Events page:
  - https://community.databricks.com/s/events
- Learning Events:
  - https://community.databricks.com/t5/learning-events/ct-p/databricks-learning-events

Ghi chú thêm:

- Community từng nhắc Learning Festival thường xuất hiện quanh các mốc `January`, `April`, `July`, `October`

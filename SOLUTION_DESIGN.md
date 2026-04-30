# GuardianEye AI: Technical Solution Design & Architectural Justification

## 1. System Overview
GuardianEye AI (GE) is an intelligent monitoring subsystem designed for real-time anomaly detection in banking transactions. It functions as an analytical module integrated into the Bank Infrastructure to provide automated risk assessment for every payment.

**Key Responsibilities:**
* **Transaction Analysis:** Real-time data ingestion from Core Banking Systems (CBS) and immediate verification via SageMaker Endpoints.
* **Verdict Generation:** Scoring (Accept/Decline/Pending) for automated decision-making.
* **Interpretability:** Calculating SHAP values for suspicious transactions to provide the Security Operations Center (SOC) with detailed risk factors.
* **MLOps Lifecycle:** Continuous performance tracking via Model Monitor and automated retraining pipelines.

## 2. Core Challenges
* **Low Latency:** Response time must remain under **100ms** to ensure seamless User Experience.
* **Extreme Throughput:** Scaling to handle **100,000 Requests Per Second (RPS)**.
* **Process Isolation:** Decoupling heavy analytical tasks (SHAP, Retraining) from the critical transaction path.
* **Data Scale:** Managing petabytes of data with a mandatory 365-day retention period.

## 3. Architectural Justification
### 3.1 Scope Isolation
GE AI is isolated from the Core Banking System (CBS) for:
* **Fault Tolerance:** A failure in the AI module must not halt the bank's core fund movements.
* **Resource Optimization:** Fraud detection requires specific GPU/Big Data resources that differ from traditional transaction processing.

### 3.2 Multi-Account Strategy
The architecture utilizes a physical separation between **Production** and **Data Science** accounts:
* **Production Account:** A restricted environment for live inference. Access is limited to automated CI/CD processes to eliminate human error.
* **Data Science Account:** A research hub for Data Scientists to experiment and train models without risking production stability.
* **Cost Management:** Differentiates **Direct Costs** (per-transaction inference) from **Overhead/R&D Costs** (training and storage).

### 3.3 Data Processing Paths (Hot, Warm, Cold)
* **Hot Path (Synchronous):** Uses AWS Lambda, SageMaker, and Online Feature Store to achieve sub-100ms latency.
* **Warm Path (Asynchronous):** Utilizes Kinesis Data Streams to process metadata, SHAP explanations, and profile updates without blocking the user.
* **Cold Path (Storage & MLOps):** Leverages S3 Data Lake and AWS Glue for long-term auditability and automated retraining cycles.

## 4. Component Deep Dive
### Account A: Production
* **API Gateway + Cognito:** Secure entry point with Client Credentials flow for banking systems.
* **Inference Lambda:** Orchestrates real-time data preparation and scoring.
* **Online Feature Store:** Low-latency access to the latest user behavioral features.
* **DynamoDB (Results):** High-performance storage for verdicts and SHAP explanations for SOC access.

### Account B: Data Science & Analytics
* **S3 Data Lake:** Centralized storage with **S3 Lifecycle Policies** for cost optimization.
* **SageMaker Model Registry:** Version control for "Approved" production-ready models.
* **Self-Healing Loop:** EventBridge triggers automated retraining via SageMaker Training Jobs when Model Monitor detects data drift.

### Governance & Security
* **IAM & KMS:** Enforcing Least Privilege and full encryption at rest/transit.
* **CloudWatch & SNS:** Real-time monitoring and instant alerting for technical and business anomalies.

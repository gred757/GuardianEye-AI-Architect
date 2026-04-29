# GuardianEye AI (GE) — Intelligent Fraud Monitoring System

## 📌 Project Overview
GuardianEye AI is an advanced monitoring subsystem designed for real-time anomaly detection in banking transactions. It handles high-load traffic (up to 100,000 RPS) with low latency while ensuring high resiliency and explainability. 

## 🏗 System Architecture
The system is built on a multi-account AWS strategy to isolate production traffic from the ML research environment. 

### Key Architecture Paths:
* **Hot Path (<100ms):** Synchronous transaction scoring using AWS Lambda, SageMaker Endpoints, and Online Feature Store. 
* **Warm Path:** Asynchronous feature updates and SHAP explainability calculation via Kinesis Data Streams and DynamoDB. 
* **Cold Path:** Long-term storage in S3 Data Lake with Glue for historical analysis and model retraining. 



## 🛠 Tech Stack
* **Cloud:** AWS (Lambda, S3, DynamoDB, Kinesis, SageMaker, EventBridge) 
* **Security & Governance:** AWS Cognito, IAM, KMS, CloudWatch 
* **Core:** Python, FastAPI, MLOps Pipelines 

## 🚀 MLOps & Resiliency
* **Automated Retraining:** Triggered by EventBridge based on Model Monitor drift alerts. 
* **Governance:** Full audit trail and encryption at rest/transit.

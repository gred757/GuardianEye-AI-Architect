# 🧱 Component Detailed Guide: GuardianEye AI

## Account A (Production)
* **API Gateway (Production) + Cognito:** High-performance entry point for requests from the Core Banking System (CBS). AWS Cognito ensures authentication for the banking system itself (via Client Credentials flow), guaranteeing that no request is processed without prior permission verification. API Gateway acts as a load balancer and a protective shield against traffic surges.
* **Lambda (Inference):** A compute module that prepares data for the model and returns the final verdict in real-time.
* **SageMaker Endpoint (Prod):** Hosting for the ML model, optimized for low latency during transaction processing.
* **Online Feature Store:** Low-latency storage for instant access to up-to-date user features.
* **Kinesis Data Streams:** A message broker for asynchronous data transmission to the Warm Path without delaying the main process.
* **DynamoDB (Results):** A high-performance NoSQL database for storing analysis results and SHAP explanations for the SOC.
* **Lambda (Feature Updater):** An asynchronous compute module that reads the transaction stream from Kinesis Data Streams and updates client profiles in the Feature Store. This ensures the model has the most current data for the next transaction.
* **Lambda (SHAP):** A specialized function for calculating risk interpretability. It uses the SHAP algorithm to identify the factors that most influenced a high suspicion score and prepares a report for security officers.
* **SageMaker Endpoint (Analytics):** A separate dedicated model instance designed for "heavy" analytical calculations like SHAP. Separation from the Prod-endpoint ensures that analytics do not slow down the main transaction flow.
* **Lambda (Result Fetcher):** A compute module acting as a backend service for the administrative panel. It fetches the required verdicts and SHAP explanations from DynamoDB (Results) upon request from the security team.
* **Kinesis Firehose:** A managed service for automatic delivery of streaming data to the S3 Data Lake. It groups small messages into larger batches and compresses them for efficient storage of petabyte-scale data.
* **API Gateway (Admin) + Cognito:** A secure gateway that accepts requests from the Security Operations Center (SOC). Combined with AWS Cognito, it provides strict authentication and authorization for security officers.

## Account B (Data Science & Analytics)
* **S3 Data Lake:** A scalable object storage for long-term retention of petabytes of data, logs, and artifacts.
* **AWS Glue Data Catalog:** A centralized metadata catalog for structuring and searching information within the Data Lake.
* **SageMaker Training:** Infrastructure for running Training Jobs to train and retrain models.
* **SageMaker Model Registry:** A repository for versioning and managing the lifecycle of approved models.
* **Model Monitor:** A system for controlling prediction quality that tracks data drift over time.
* **AWS CodePipeline:** An automation tool (CI/CD) for secure deployment of new model versions from Account B to Account A.
* **S3 Lifecycle Policies:** A set of rules for the automatic movement and deletion of data to optimize storage costs.
* **EventBridge Train (Drift Trigger):** An intelligent trigger that reacts to critical alerts from Model Monitor. It automatically initiates a new SageMaker Training Job to retrain the model on fresh data if quality falls below a set threshold.
* **EventBridge Scheduler:** A scheduler that ensures the execution of routine maintenance, triggering model retraining according to the bank's schedule.
* **EventBridge Deploy:** A control element that links the development stage to production. Once a new model in the Model Registry is "Approved", this trigger launches AWS CodePipeline for a secure update in Account A.

## Governance & Monitoring Layer
* **IAM (Identity and Access Management):** Enforces the "Least Privilege" principle, granting services access only to necessary resources.
* **KMS (Key Management Service):** Manages encryption keys to ensure data in S3, DynamoDB, and Kinesis is encrypted "at rest" and during transit.
* **CloudWatch:** A central monitoring system that collects metrics and triggers alerts for self-healing or retraining processes.
* **SNS (Simple Notification Service):** An instant notification service used to inform technical teams or security officers about critical events.

## External Layer (Bank Infrastructure)
* **CBS (Core Banking System / Source of Truth):** The primary banking system and initiator of verification requests; serves as the "Source of Truth" for GE AI.
* **Security Operations Center (SOC):** The bank's security team that uses GE AI reports for final confirmation or rejection of suspicious operations.
* **User:** An external entity initiating a transaction; interacts solely with the banking interface, unaware of the AI processes.

# [MLOps] Ephemeral AI Environments on Pull Request

[![MLOps](https://img.shields.io/badge/MLOps-Architecture-blueviolet)](https://github.com/your-username/your-repo)
[![AWS](https://img.shields.io/badge/AWS-SageMaker-orange)](https://aws.amazon.com/sagemaker/)
[![Terraform](https://img.shields.io/badge/Infrastructure-as-Code-blue)](https://www.terraform.io/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub--Actions-black)](https://github.com/features/actions)

> **I’d like to share an automated MLOps CI/CD workflow that provisions temporary AI environments the moment a Pull Request (PR) is opened.**

---

## 📌 1. Problem Statement: The Challenges of Traditional Staging
In a standard MLOps workflow, the Staging environment serves as the final "buffer" before Production deployment. However, the traditional approach of maintaining a permanent, static Staging environment reveals two critical flaws:

* **Cost Inefficiency:** High-performance resources, such as SageMaker Endpoints, incur significant costs if maintained 24/7, even when no testing activities are occurring.
* **Version Congestion:** When multiple developers work simultaneously, sharing a single static Staging environment makes it difficult to isolate specific features for independent validation.

## 🏗️ 2. Solution Architecture: Ephemeral AI Environments
This solution shifts toward **On-demand Infrastructure**. The system is event-driven and fully automated using modern DevOps and Cloud tools.

### A. Continuous Integration (CI) Automation
The moment a Pull Request (PR) is initiated on **GitHub**, the CI pipeline executes:
1.  **Containerization:** Packaging source code, dependencies, and Model Artifacts into a Docker Image.
2.  **Registry Storage:** Pushing the image to **Amazon ECR**. This ensures **Immutability**—the exact image tested is the one that will eventually reach Production.

### B. Infrastructure as Code (IaC)
Utilizing **Terraform** to define the infrastructure, the system automatically provisions:
* **Amazon SageMaker Endpoint:** Provides compute power for real-time model inference.
* **Amazon API Gateway:** Acts as an abstraction layer, providing a single RESTful API for reviewers to interact with without requiring direct AWS Console access.

## 🔄 3. Operational Workflow & Validation
The core value lies in the **Real-time Feedback Loop**:
* **Inference Testing:** Reviewers (or automated scripts) send actual requests through the API Gateway.
* **Immediate Prediction:** The model responds instantly within a production-like environment, allowing for an accurate assessment of **latency**, **data schema compatibility**, and **logic accuracy**.

## 🧹 4. Cleanup & Resource Optimization
This is the most crucial phase for cost management. An **Auto-Destroy** process is triggered as soon as the PR is **Merged** or **Closed**:
* **Terraform Destroy:** Executes the `destroy` command to reclaim all allocated AWS resources.
* **Efficiency:** Resource uptime matches the review duration perfectly. This can reduce staging operational costs by **70% to 90%**.

## 🚀 5. Conclusion
Implementing **Ephemeral AI Environments** is not just a technical upgrade but a paradigm shift in resource management for MLOps. It instills confidence in development teams, ensuring the highest model quality while maintaining financial agility and operational efficiency.

---

### 🛠️ Technology Stack
| Component | Technology |
| :--- | :--- |
| **CI/CD Platform** | GitHub Actions |
| **Cloud Provider** | AWS (Amazon Web Services) |
| **Model Hosting** | Amazon SageMaker |
| **IaC Tool** | Terraform |
| **Registry** | Amazon ECR |
| **Gateway** | Amazon API Gateway |

---

### 📖 How to Use
1.  **Open a Pull Request** with your model or code changes.
2.  **GitHub Actions** will trigger the Terraform plan/apply.
3.  Once the endpoint is live, perform your **Inference Testing**.
4.  **Merge or Close** the PR to trigger the automatic cleanup of all AWS resources.

---

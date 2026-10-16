# Data Ingestion Strategy Recommendation for QuickCart

**To:** Chief Technology Officer (CTO)  
**From:** Junior Data Engineer  
**Date:** October 14, 2026  
**Subject:** Technical & Financial Evaluation for Data Ingestion Platform (Fivetran vs. Airbyte OSS)

---

### 1. Executive Summary
To build QuickCart’s initial data stack, we evaluated two primary ingestion solutions—**Fivetran** (Fully Managed SaaS) and **Airbyte Open Source** (Self-Hosted on Kubernetes). Based on our high data volume, custom internal APIs, tight software budget, and existing cloud credits, **we strongly recommend adopting Airbyte Open Source (OSS)**.

While Airbyte requires self-hosting maintenance, it eliminates prohibitive monthly software fees and seamlessly integrates with our custom systems.

---

### 2. Constraints & Comparative Analysis

| Evaluation Metric | Fivetran (SaaS) | Airbyte Open Source (OSS) | Impact on QuickCart |
| :--- | :--- | :--- | :--- |
| **High Volume Source (80M MAR/Month)** | **Extremely Expensive:** Charges based on Monthly Active Rows (MAR). 80M rows will cause severe billing shocks. | **Free Software License:** Cost is strictly tied to infrastructure compute. | **Airbyte Wins:** We leverage unused AWS EC2/Kubernetes hosting credits at $0 extra licensing cost. |
| **Custom Inventory API (No Docs)** | **Infeasible:** Black-box platform with no capability to build custom connectors for private internal APIs. | **Fully Supported:** Features a low-code Connector Builder and Python CDK to build connectors in minutes. | **Airbyte Wins:** Enables us to quickly connect our undocumented in-house inventory API. |
| **Financial Constraints** | High, unpredictable recurring SaaS operational expenses. | Tight immediate software budget, but offset by available AWS credits. | **Airbyte Wins:** Protects runway by utilizing pre-allocated cloud credits. |
| **Operational Overhead** | Zero-maintenance. | Requires infrastructure management (Helm, Docker, K8s). | **Fivetran Wins:** Lowers burden on our 1 Senior DE. |

---

### 3. Risk Mitigation & Implementation Plan for Airbyte OSS
Because our engineering team is small (1 Senior DE & 1 Junior DE), running Airbyte OSS introduces operational risks that we must actively mitigate:

1. **Preventing Out-of-Memory (OOM) Crashes:**
   * **Risk:** Heavy initial syncs on PostgreSQL (80M rows) can crash Docker pods.
   * **Fix:** Tune JVM heap sizes, configure explicit memory limits on worker pods, and use Change Data Capture (CDC) via PostgreSQL WAL logs for incremental loads.
2. **Avoiding Disk Storage Inflation:**
   * **Risk:** Verbose Airbyte logs can quickly fill up server storage and crash metadata databases.
   * **Fix:** Configure strict automated log rotation and 7-day log retention policies on EC2/Kubernetes.
3. **Custom API Integration:**
   * **Action:** I (Junior DE) will utilize Airbyte’s Connector Builder to reverse-engineer and map the undocumented internal inventory API endpoints into a production-grade connector.

---

### 4. Final Recommendation & Next Steps

* **Final Decision:** Deploy **Airbyte Open Source** on our existing AWS Kubernetes (EKS) cluster.
* **Immediate Next Steps:**
  1. Provision an Airbyte instance using Helm chart on our unused AWS Kubernetes cluster.
  2. Configure CDC replication on the PostgreSQL production database to handle daily batch syncs efficiently.
  3. Build and test the custom Airbyte Connector for the internal inventory API.
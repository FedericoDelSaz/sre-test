## **🥜 Proposed Architecture for the Document Processor System**

| **Challenge**                  | **Proposed Solution**                                                                                                                                                                                                                                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Scalability & Performance**  | ✅ Migrate to **AWS EKS** for Kubernetes-based auto-scaling (**Karpenter**) <br> ✅ Use **RabbitMQ** for asynchronous processing |
| **Traffic Control & Routing**  | ✅ Use **Amazon Route 53** for **failover & latency-based routing** |
| **Storage & Data Security**    | ✅ Store documents in **Amazon S3** for processed data <br> ✅ Use **Amazon EFS** for raw image processing <br> ✅ **Encrypt documents** for security and compliance |
| **Security & Compliance**      | ✅ Implement **Kong API Gateway** with **OAuth plugin** for authentication & rate limiting <br> ✅ Enhance security using **Istio Service Mesh** for service-to-service communication <br> ✅ Use **Kyverno** for policy enforcement and **AWS Secret Manager** for secrets management |
| **Reliability & Fault Tolerance** | ✅ Use **Route 53 failover** for high availability <br> ✅ Implement **AWS Backup Service** for S3 & EFS backups |
| **Deployment & Release Risks** | ✅ Adopt **Canary & Blue-Green Deployments** using **ArgoCD** for GitOps-driven deployments |
| **Containerization & Portability** | ✅ **Dockerize applications** for consistency across environments <br> ✅ Deploy containers using **Amazon EKS** for managed **Kubernetes orchestration** |
| **Service Mesh & Observability** | ✅ Integrate **Istio Service Mesh** for traffic management, security, and observability <br> ✅ Implement **full observability** using **ELK, Instana, Prometheus, and Grafana** |

---

### **📞 Architecture Diagram**

#### **Approach 1 (Single EKS Cluster with Istio Service Mesh)**

```mermaid
graph LR
  subgraph "Mobile App"
  Drivers["Driver"]
  end
  subgraph "Desktop App"
  Agents["Customer Service Agents"]
  end
  subgraph "AWS Cloud"
    subgraph "Networking"
      Route53["Amazon Route 53"]
      ALB["ALB/NLB"]
    end
    subgraph "Security"
      Kong["Kong API Gateway"]
      Istio["Istio Service Mesh"]
    end
    subgraph "Compute"
      EKS["AWS EKS (Kubernetes Cluster)"]
      DocService["Document Processor Service"]
    end
    subgraph "Storage"
      EFS["Amazon EFS (Raw Processing)"]
      S3["Amazon S3 (Processed Documents)"]
      MongoDB["MongoDB (Metadata)"]
      AWSBackup["AWS Backup Service (EFS & S3)"]
    end
  end

  Drivers-- "Browse and Upload Documents API" -->Route53
  Agents-- "Browse and Administer Documents API" -->Route53
  Route53 --> ALB
  ALB --> Kong
  Kong --> Istio
  Istio --> EKS
  EKS --> DocService
  DocService-- "Process, Encrypt, Store" --> EFS
  DocService-- "Store Processed Document in S3" --> S3
  DocService-- "Store Metadata in MongoDB" --> MongoDB
  EFS-- "Backups to" --> AWSBackup
  S3-- "Backups to" --> AWSBackup
```

#### **Approach 2 (Dual EKS Clusters per Sprint with Istio Service Mesh)**

```mermaid
graph LR
  subgraph "Mobile App"
    Drivers["Driver"]
  end
  subgraph "Desktop App"
    Agents["Customer Service Agents"]
  end
  subgraph "AWS Cloud"
    subgraph "Networking"
      Route53["Amazon Route 53 (Failover & Latency Routing)"]
      ALB["ALB/NLB"]
    end
    subgraph "Security"
      Kong["Kong API Gateway"]
      Istio["Istio Service Mesh"]
    end
    subgraph "Compute"
      subgraph "Europe Region"
        EKS1["AWS EKS - Even Sprint Cluster"]
        EKS2["AWS EKS - Odd Sprint Cluster"]
      end
      DocService1["Document Processor Service (Even Sprint)"]
      DocService2["Document Processor Service (Odd Sprint)"]
    end
    subgraph "Storage"
      EFS["Amazon EFS (Raw Processing)"]
      S3["Amazon S3 (Processed Documents)"]
      MongoDB["MongoDB (Metadata)"]
      AWSBackup["AWS Backup Service (EFS & S3)"]
    end
  end

  Drivers-- "Browse and Upload Documents API" -->Route53
  Agents-- "Browse and Administer Documents API" -->Route53
  Route53 --> ALB
  ALB --> Kong
  Kong --> Istio
  Istio --> EKS1 & EKS2
  EKS1 --> DocService1
  EKS2 --> DocService2
  DocService1 & DocService2-- "Process, Encrypt, Store" --> EFS
  DocService1 & DocService2-- "Store Processed Document in S3" --> S3
  DocService1 & DocService2-- "Store Metadata in MongoDB" --> MongoDB
  EFS-- "Backups to" --> AWSBackup
  S3-- "Backups to" --> AWSBackup
```

---

### **📊 Feature Comparison**

| **Feature**                     | **Approach 1 (Single EKS Cluster with Istio)** | **Approach 2 (Dual EKS Clusters with Istio per Sprint)** |
|----------------------------------|--------------------------------------------|---------------------------------------------|
| **Traffic Control**              | ✅ Amazon Route 53 (low cost)              | ✅ Amazon Route 53 (low cost)               |
| **Load Balancing**               | ✅ AWS ALB/NLB (scalable, pay-as-you-go) | ✅ AWS ALB/NLB (scalable, pay-as-you-go)   |
| **Security**                     | ✅ Kong API Gateway with OAuth (moderate cost) | ✅ Kong API Gateway with OAuth (higher due to 2 clusters) |
| **Compute (Execution Environment)** | ✅ Single AWS EKS Cluster (scalable, efficient cost) | ✅ Two AWS EKS Clusters (double EKS cost) |
| **Deployment Strategy**          | ✅ ArgoCD + Argo Rollouts (low additional cost) | ✅ ArgoCD (low cost, no Argo Rollouts) |
| **Scalability (Karpenter)** | ✅ Kubernetes-based auto-scaling (efficient cost) | ✅ Kubernetes-based auto-scaling (higher cost due to 2 clusters) |
| **Async Processing**             | ✅ RabbitMQ (low cost per message processed) | ✅ Separate RabbitMQ per cluster (higher due to duplication) |
| **Observability**                | ✅ ELK, Instana, Prometheus, Grafana       | ✅ ELK, Instana, Prometheus, Grafana       |
| **Cost Consideration 💰**        | 💲 **Optimized (Single EKS, scalable, moderate cost)** | 💲💲 **Higher (2x EKS clusters, better isolation)** |

---


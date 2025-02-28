## **🦜 Proposed Architecture for the Document Processor System**

| **Challenge**                  | **Proposed Solution**                                                                                                                                                                                                                                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Scalability & Performance**  | ✅ Migrate to **AWS EKS** for Kubernetes-based auto-scaling (**Karpenter**) <br> ✅ Use **RabbitMQ** for asynchronous processing |
| **Traffic Control & Routing**  | ✅ Use **Amazon Route 53** for **failover & latency-based routing** |
| **Storage & Data Security**    | ✅ Store documents in **Amazon S3** (scalable, low latency) <br> ✅ **Encrypt documents** for security and compliance |
| **Security & Compliance**      | ✅ Implement **Kong API Gateway** for authentication & rate limiting |
| **Reliability & Fault Tolerance** | ✅ Use **Route 53 failover** for high availability <br> ✅ Implement **offsite backups** in **AWS S3/Glacier** |
| **Deployment & Release Risks** | ✅ Adopt **Canary & Blue-Green Deployments** for safer rollouts and minimal downtime |
| **Containerization & Portability** | ✅ **Dockerize applications** for consistency across environments <br> ✅ Deploy containers using **Amazon EKS** for managed **Kubernetes orchestration** |
| **Service Mesh & Observability** | ✅ Integrate **Istio Service Mesh** for traffic management, security, and observability |

---

### **📞 Architecture Diagram**

#### **Approach 1 (Single EKS Cluster with Istio Service Mesh)**
```mermaid
graph TD;
  
  subgraph AWS_Cloud
    subgraph Networking
      A[Amazon Route 53] -->|Failover & Latency Routing| B[ALB/NLB]
    end
    
    subgraph Security
      C[Kong API Gateway] -->|Auth, Rate Limiting| B
    end

    subgraph Compute
      F[AWS EKS (Kubernetes Cluster)] -->|Containerized Backend| G[Dockerized Microservices]
      G -->|Async Processing| H[RabbitMQ]
      G -->|Service Mesh| I[Istio Service Mesh]
    end

    subgraph Storage
      J[AWS S3] -->|Encrypted Storage| K[Data Lake / Documents]
      K -->|Backups| L[AWS Glacier]
    end

    subgraph Deployment
      M[CI/CD Pipeline] -->|Triggers Deployment| N[ArgoCD]
      N -->|Manages Rollouts| O[Argo Rollouts]
      O -->|Progressive Delivery| F
    end
  end
  
  A -->|Traffic Control| C
  B --> F
  F --> J
```

#### **Approach 2 (Dual EKS Clusters per Sprint with Istio Service Mesh)**
```mermaid
graph TD;
  
  subgraph AWS_Cloud
    subgraph Networking
      A[Amazon Route 53] -->|Failover & Latency Routing| B[ALB/NLB]
    end
    
    subgraph Security
      C[Kong API Gateway] -->|Auth, Rate Limiting| B
    end

    subgraph Compute
      subgraph Europe_Region
        F1[AWS EKS - Even Sprint Cluster] -->|Containerized Backend| G1[Dockerized Microservices (Even)]
        F2[AWS EKS - Odd Sprint Cluster] -->|Containerized Backend| G2[Dockerized Microservices (Odd)]
        G1 -->|Async Processing| H1[RabbitMQ (Even)]
        G2 -->|Async Processing| H2[RabbitMQ (Odd)]
        G1 -->|Service Mesh| I1[Istio Service Mesh (Even)]
        G2 -->|Service Mesh| I2[Istio Service Mesh (Odd)]
      end
    end

    subgraph Storage
      J[AWS S3] -->|Encrypted Storage| K[Data Lake / Documents]
      K -->|Backups| L[AWS Glacier]
    end

    subgraph Deployment
      M[CI/CD Pipeline] -->|Triggers Deployment| N[ArgoCD]
      N -->|Deploys to Even Sprint Cluster| F1
      N -->|Deploys to Odd Sprint Cluster| F2
    end
  end
  
  A -->|Traffic Control| C
  B -->|Routes Traffic| F1 & F2
  F1 & F2 --> J
```

---

### **📊 Feature Comparison**
| **Feature**                     | **Approach 1 (Single EKS Cluster with Istio)** | **Approach 2 (Dual EKS Clusters with Istio per Sprint)** |
|----------------------------------|--------------------------------------------|---------------------------------------------|
| **Traffic Control**              | ✅ Amazon Route 53 (low cost)              | ✅ Amazon Route 53 (low cost)               |
| **Load Balancing**               | ✅ AWS ALB/NLB (scalable, pay-as-you-go) | ✅ AWS ALB/NLB (scalable, pay-as-you-go)   |
| **Security**                     | ✅ Kong API Gateway (moderate cost) | ✅ Kong API Gateway (higher due to 2 clusters) |
| **Compute (Execution Environment)** | ✅ Single AWS EKS Cluster (scalable, efficient cost) | ✅ Two AWS EKS Clusters (double EKS cost) |
| **Deployment Strategy**          | ✅ ArgoCD + Argo Rollouts (low additional cost) | ✅ ArgoCD (low cost, no Argo Rollouts) |
| **Scalability (Karpenter)** | ✅ Kubernetes-based auto-scaling (efficient cost) | ✅ Kubernetes-based auto-scaling (higher cost due to 2 clusters) |
| **Sprint-Based Deployment**      | ❌ Single cluster handles all sprints     | ✅ One EKS for even sprints, one for odd (higher infra cost) |
| **Async Processing**             | ✅ RabbitMQ (low cost per message processed) | ✅ Separate RabbitMQ per cluster (higher due to duplication) |
| **Backup Strategy**              | ✅ AWS S3 + Glacier backups (low cost)    | ✅ AWS S3 + Glacier backups (low cost) |
| **High Availability (HA)**       | ✅ Kubernetes redundancy & failover       | ✅ Multi-cluster redundancy (even/odd) (higher cost) |
| **Resilience & Fault Tolerance** | ✅ Kubernetes cluster self-healing        | ✅ Two independent clusters for resilience |
| **Service Mesh (Istio)**         | ✅ Centralized traffic control, observability, security | ✅ Separate Istio instances for each cluster (higher cost) |
| **Cost Consideration 💰**        | 💲 **Optimized (Single EKS, scalable, moderate cost)** | 💲💲 **Higher (2x EKS clusters, better isolation)** |

---

### **🚀 Recommended Approach**
- **💰 Best cost-performance balance** → **Approach 1 (Single EKS Cluster with Istio).**
- **🤖 Need structured sprint-based testing?** → **Approach 2 (Dual EKS Clusters with Istio).**
- **🌟 Want to maximize cost savings?**
    - **Use Approach 2** but **shutdown the Even EKS** after traffic shifts to Odd.


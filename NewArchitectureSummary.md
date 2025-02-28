## **🚀 Proposed Architecture for the Document Processor System**

### **Key Components**
- **AWS EKS (Kubernetes) for containerized microservices**
- **Amazon Route 53 for traffic routing**
- **Kong API Gateway for security & OAuth & rate limiting**
- **RabbitMQ for async processing**
- **AWS EFS (encrypted) + AWS Glacier for storage & backups**
- **ArgoCD + Argo Rollouts for CI/CD deployments**
- **Karpenter for dynamic node scaling & cost efficiency**
- **Istio - Service Mesh**
- **Kyverno - Cluster Policy**


## **Approach 1: Single EKS Cluster**
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
      F[AWS EKS (Single Cluster)]
      F -->|Containerized Backend| G[Dockerized Microservices]
      G -->|Async Processing| H[RabbitMQ]
      F -->|Dynamic Node Scaling| Karpenter[Karpenter Auto-Scaling]
    end

    subgraph Storage
      I[AWS S3] -->|Encrypted Storage| J[Data Lake / Documents]
      J -->|Backups| Glacier[AWS Glacier]
    end

    subgraph Deployment
      L[CI/CD Pipeline] -->|Triggers Deployment| M[ArgoCD]
      M -->|Manages Rollouts| N[Argo Rollouts]
      N -->|Progressive Delivery| F
    end
  end
  
  A -->|Traffic Control| C
  C --> B
  B --> F
  F --> I


```

---

## **Approach 2: Dual EKS Clusters (Even/Odd Sprints)**
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
        F1 & F2 -->|Dynamic Node Scaling| K[Karpenter Auto-Scaling]
      end
    end

    subgraph Storage
      I[AWS S3] -->|Encrypted Storage| J[Data Lake / Documents]
      J -->|Backups| K[AWS Glacier]
    end

    subgraph Deployment
      L[CI/CD Pipeline] -->|Triggers Deployment| M[ArgoCD]
      M -->|Deploys to Even Sprint Cluster| F1
      M -->|Deploys to Odd Sprint Cluster| F2
    end
  end
  
  A -->|Traffic Control| C
  B -->|Routes Traffic| F1 & F2
  F1 & F2 --> I
```

---

## **🏆 Recommended Strategy**
- **For cost-performance balance** → **Approach 1 (Single EKS Cluster)**.
- **For structured, sprint-based testing & risk mitigation** → **Approach 2 (Dual EKS Clusters + Traffic Weight Shifting).**

Approach 2 is **preferred for production** if **beta validation before rollout** is a priority.


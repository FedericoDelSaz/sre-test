## **🚀 Proposed Architecture for the Document Processor System**
### **Key Components**
- **AWS EKS (Kubernetes) for containerized microservices**
- **Amazon Route 53 for traffic routing**
- **Kong API Gateway + AWS WAF + AWS Shield for security**
- **RabbitMQ for async processing**
- **AWS S3 (encrypted) + AWS Glacier for storage & backups**
- **ArgoCD + Argo Rollouts for CI/CD deployments**
- **Karpenter for dynamic node scaling & cost efficiency**

---

## **🔄 Approach Comparison: Single vs. Dual EKS Clusters**
| **Feature**                     | **Approach 1: Single EKS Cluster**  | **Approach 2: Dual EKS Clusters (Even/Odd Sprints)** |
|----------------------------------|-------------------------------------|-----------------------------------------------------|
| **Traffic Control**              | ✅ Route 53 (Failover & Latency Routing) | ✅ Route 53 (Weighted Traffic Shifting) |
| **Security**                     | ✅ Kong API Gateway + AWS WAF + Shield | ✅ Kong API Gateway + AWS WAF + Shield (2 clusters) |
| **Scalability**                   | ✅ Kubernetes auto-scaling + Karpenter | ✅ Kubernetes auto-scaling + Karpenter |
| **Deployment Strategy**          | ✅ ArgoCD + Argo Rollouts | ✅ ArgoCD (No Rollouts, but structured traffic shifting) |
| **Sprint-Based Testing**         | ❌ Limited to feature flags | ✅ Separate cluster for beta testing |
| **Cost Efficiency**              | ✅ Lower operational cost | ⚠️ Higher infra cost but optimized by shutting down unused cluster |

---

## **Approach 1: Single EKS Cluster**
```mermaid
graph TD;
  
  subgraph AWS_Cloud
    subgraph Networking
      A[Amazon Route 53] -->|Failover & Latency Routing| B[ALB/NLB]
    end
    
    subgraph Security
      C[Kong API Gateway] -->|Auth, Rate Limiting| D[AWS WAF]
      D -->|DDoS Protection| E[AWS Shield]
    end

    subgraph Compute
      F[AWS EKS (Single Cluster)] -->|Containerized Backend| G[Dockerized Microservices]
      G -->|Async Processing| H[RabbitMQ]
      F -->|Dynamic Node Scaling| K[Karpenter Auto-Scaling]
    end

    subgraph Storage
      I[AWS S3] -->|Encrypted Storage| J[Data Lake / Documents]
      J -->|Backups| K[AWS Glacier]
    end

    subgraph Deployment
      L[CI/CD Pipeline] -->|Triggers Deployment| M[ArgoCD]
      M -->|Manages Rollouts| N[Argo Rollouts]
      N -->|Progressive Delivery| F
    end
  end
  
  A -->|Traffic Control| C
  B --> F
  C --> B
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
      C[Kong API Gateway] -->|Auth, Rate Limiting| D[AWS WAF]
      D -->|DDoS Protection| E[AWS Shield]
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
  C --> B
  F1 & F2 --> I

```

---

## **🏆 Recommended Strategy**
- **For cost-performance balance** → **Approach 1 (Single EKS Cluster)**.
- **For structured, sprint-based testing & risk mitigation** → **Approach 2 (Dual EKS Clusters + Traffic Weight Shifting).**

Approach 2 is **preferred for production** if **beta validation before rollout** is a priority.
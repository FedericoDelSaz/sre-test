## **🚀 HA & Migration Plan**

### **🔹 Phase 1: High Availability (HA) Setup**
✅ Route 53, AWS WAF & Shield for traffic management and security.  
✅ Duplicate Document Processor VM & Load Balancer for HA.  
✅ Zero Downtime Rollouts using Blue-Green Deployment.

### **🔹 Phase 2: Storage Migration**
✅ Migrate NFS to S3 with lifecycle rules and backups.

### **🔹 Phase 3: Containerization**
✅ Dockerize applications while keeping them in VMs.

### **🔹 Phase 4: Migration to EKS (Even Part Only)**
✅ Deploy **first EKS cluster + Kong API Gateway**.

### **🔹 Phase 5: **Monitoring & Logging Implementation**
✅ Deploy **ELK (or OpenSearch) for log aggregation**.  
✅ Set up **Instana** for performance monitoring.  
✅ Enable **Kubernetes native monitoring** (Prometheus + Grafana).

### **🔹 Phase 6: Kubernetes Optimization**
✅ Implement:
- **Karpenter** (autoscaling)
- **ArgoCD** (GitOps-based CI/CD)
- **Kyverno** (policy enforcement)
- **Secret Manager** (secure credentials)
- **Cert Manager** (TLS certificates)

### **🔹 Phase 7: Full EKS Migration (Add Part)**
✅ Migrate remaining workloads to **EKS + Kong**.

### **🔹 Phase 8: Advanced Monitoring & Security**
✅ Fine-tune **ELK/OpenSearch + Instana + Prometheus + Grafana**.  
✅ Improve security policies via Kyverno & AWS Shield.

### **🔹 Phase 9: Dynamic EKS Cluster Management**
✅ Develop **mechanism to shut down and start EKS clusters dynamically** to optimize costs.

---

## **🗺️ Updated EKS & Monitoring Architecture**
```mermaid
graph LR
  subgraph "AWS Cloud"
    R53["Amazon Route 53"]
    WAF["AWS WAF"]
    Shield["AWS Shield"]
  end

  subgraph "Load Balancer (HA)"
    LB1["Primary Load Balancer"]
    LB2["Secondary Load Balancer (Failover)"]
  end

  subgraph "Document Processor (VMs & EKS)"
    VM1["Document Processor VM (Active)"]
    VM2["Document Processor VM (Passive)"]
    EKS1["EKS Workloads (Even Part)"]
    EKS2["EKS Workloads (Add Part)"]
  end

  subgraph "Storage Lifecycle"
    S3_Std["S3 Standard (Active Docs)"]
    S3_IA["S3 Infrequent Access (Older Docs)"]
    Glacier["S3 Glacier (Archived Docs)"]
  end

  subgraph "Backup & DR"
    BackupS3["S3 Backup Bucket"]
    DRRegion["Cross-Region Replication"]
  end

  subgraph "Kubernetes Tooling"
    Karpenter["Karpenter (Autoscaling)"]
    ArgoCD["ArgoCD (CI/CD)"]
    Kyverno["Kyverno (Policy)"]
    Secrets["Secret Manager"]
    Certs["Cert Manager"]
  end

  subgraph "Monitoring & Logging"
    Instana["Instana (Performance Monitoring)"]
    ELK["ELK / OpenSearch Stack"]
    Prometheus["Prometheus + Grafana"]
  end

  R53 -->|Routes Traffic| WAF
  WAF -->|DDoS & Security Filtering| Shield
  Shield -->|Weighted Routing| LB1
  Shield -->|Failover Routing| LB2
  LB1 --> VM1
  LB2 --> VM2
  VM1 --> EKS1
  VM2 --> EKS2
  EKS1 --> Karpenter
  EKS1 --> ArgoCD
  EKS1 --> Kyverno
  EKS1 --> Secrets
  EKS1 --> Certs
  EKS1 --> Instana
  EKS1 --> ELK
  EKS1 --> Prometheus
  EKS2 --> Karpenter
  EKS2 --> ArgoCD
  EKS2 --> Kyverno
  EKS2 --> Secrets
  EKS2 --> Certs
  EKS2 --> Instana
  EKS2 --> ELK
  EKS2 --> Prometheus
  VM1 -- "Stores Files" --> S3_Std
  VM2 -- "Stores Files" --> S3_Std
  S3_Std -- "Lifecycle Rule" --> S3_IA
  S3_IA -- "Archival after X Days" --> Glacier
  S3_Std -- "Daily Backups" --> BackupS3
  BackupS3 -- "Disaster Recovery" --> DRRegion
```

---

## **🔹 Key Benefits**
✅ **Zero Downtime** via HA Load Balancers & Blue-Green Strategy.  
✅ **Scalable Storage** using S3 with lifecycle policies.  
✅ **Full Observability** with **Instana, ELK/OpenSearch, Prometheus & Grafana**.  
✅ **Seamless Migration** from VMs → EKS with phased rollout.  
✅ **Cost Optimization** via auto-scaling (Karpenter) & EKS shutdown mechanism.  
✅ **Enhanced Security** via AWS WAF, Shield, Kyverno & Secret Manager.

---

## **🚀 HA & Migration Plan**

### **🔹 Phase 1: High Availability (HA) Setup**

✅ Route 53 for traffic management.\
✅ Duplicate Document Processor VM & Load Balancer for HA.\
✅ Zero Downtime Rollouts using Blue-Green Deployment.

### **🔹 Phase 2: Storage Migration**

✅ Migrate NFS to EFS with lifecycle rules and backups.

### **🔹 Phase 3: Containerization**

✅ Dockerize applications while keeping them in VMs.

### **🔹 Phase 4: Migration to EKS (Even Part Only)**

✅ Deploy **first EKS cluster + Kong API Gateway**.\
✅ Kong OAuth and Rate Limit plugins.

### **🔹 Phase 5: Kubernetes Optimization (Even Part Only)**

✅ Implement:

- **Karpenter** (autoscaling)
- **ArgoCD** (GitOps-based CI/CD)
- **Kyverno** (policy enforcement)
- **Secret Manager** (secure credentials)
- **Cert Manager** (TLS certificates)
- **Istio** (Service Mesh)

### **🔹 Phase 6: Deploy Applications in EKS (Even Part Only)**

✅ Migrate applications to Helm charts.

✅ Deploy Amazon MQ\
✅ Implement CI (GitHub Actions) & CD (ArgoCD):

- ✅ Document Processor
- ✅ Desktop App
- ✅ Document Service

### **🔹 Phase 7: Monitoring & Logging Implementation (Even Part Only)**

✅ Deploy **ELK (or OpenSearch) for log aggregation**.\
✅ Set up **Instana** for performance monitoring.\
✅ Enable **Kubernetes native monitoring** (Prometheus + Grafana).

### **🔹 Phase 8: Full EKS Migration (Odd Part)**

✅ Migrate remaining workloads to **EKS + Kong**.\
✅ Kong OAuth and Rate Limit plugins.

### **🔹 Phase 9: Kubernetes Optimization (Odd Part)**

✅ Implement:

- **Karpenter** (autoscaling)
- **ArgoCD** (GitOps-based CI/CD)
- **Kyverno** (policy enforcement)
- **Secret Manager** (secure credentials)
- **Cert Manager** (TLS certificates)
- **Istio** (Service Mesh)

### **🔹 Phase 10: Dynamic EKS Cluster Management**

✅ Develop **mechanism to shut down and start EKS clusters dynamically** to optimize costs. We can use Karpenter (NodePools).

---

## **🔹 Key Benefits**

✅ **Zero Downtime** via HA AWS Route 53 & Blue-Green Strategy.\
✅ **Scalable Storage** using EFS with lifecycle policies.\
✅ **Full Observability** with **Instana, ELK/OpenSearch, Prometheus & Grafana**.\
✅ **Seamless Migration** from VMs → EKS with phased rollout.\
✅ **Cost Optimization** via auto-scaling (Karpenter) & EKS shutdown mechanism.\
✅ **Enhanced Security** with Istio, Kong OAuth and rate limit plugins, Kyverno & Secret Manager.


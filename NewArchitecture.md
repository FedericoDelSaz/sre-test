## **🦜 Proposed Architecture for the Document Processor System**

| **Challenge**                  | **Proposed Solution**                                                                                                                                                                                                                                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Scalability & Performance**  | ✅ Migrate to **AWS EKS** for Kubernetes-based auto-scaling (**Karpenter**) <br> ✅ Use **Amazon MQ** for asynchronous processing |
| **Traffic Control & Routing**  | ✅ Use **Amazon Route 53** for **failover & latency-based routing** |
| **Storage & Data Security**    | ✅ Store documents in **Amazon EFS** (scalable, low latency) <br> ✅ **Encrypt documents** for security and compliance <br> ✅ Store document URLs in **MongoDB Atlas** for easy access and retrieval of metadata |
| **Security & Compliance**      | ✅ Implement **Kong API Gateway** for authentication & rate limiting |
| **Reliability & Fault Tolerance** | ✅ Use **Route 53 failover** for high availability <br> ✅ Implement **AWS Backup Service** for EFS backups |
| **Deployment & Release Risks** | ✅ Adopt **Canary & Blue-Green Deployments** for safer rollouts and minimal downtime |
| **Containerization & Portability** | ✅ **Dockerize applications** for consistency across environments <br> ✅ Deploy containers using **Amazon EKS** for managed **Kubernetes orchestration** |
| **Service Mesh & Observability** | ✅ Integrate **Istio Service Mesh** for traffic management, security, and observability |
| **Mobile Notifications**       | ✅ Add **NotificationService** to send notifications to the mobile app via **Firebase Cloud Messaging (FCM)** |

---

### **📞 Architecture Diagram**

#### **Single EKS Cluster with Istio Service Mesh**

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
    end
    subgraph "Compute"
      EKS["AWS EKS (Kubernetes Cluster)"]
      DocService["Document Processor Service"]
      DocStorage["Document Storage Service"]
      NotificationService["Notification Service"]
      AmazonMQ["Amazon MQ (RabbitMQ)"]
      UserService["User Service"]
    end
    subgraph "Storage"
      EFS["Amazon EFS (Encrypted)"]
      MongoDB["MongoDB (User Data & Download URLs)"]
      AWSBackup["AWS Backup Service (EFS)"]
      Redis["Redis (User Cache)"]
    end
  end

  Drivers-- "Browse and Upload Documents API" -->Route53
  Agents-- "Browse and Administer Documents API" -->Route53
  Route53 --> ALB
  ALB --> Kong
  Kong --> EKS
  EKS --> DocService
  EKS --> UserService
  DocService-- "Queue Processing" --> AmazonMQ
  DocStorage-- "Listening for Messages" --> AmazonMQ
  DocStorage-- "Process, Encrypt, Store" --> EFS
  DocStorage-- "Store Encrypted Download URL" --> MongoDB
  DocStorage-- "Send Push Notification with Download URL" --> NotificationService
  NotificationService-- "Send Push Notifications" --> Drivers
  EFS-- "Backups to" --> AWSBackup
  UserService-- "Store User Data" --> MongoDB
  UserService-- "Cache User Data" --> Redis


```

---

### **Process Flow**

1. **Drivers & Agents** interact with the system through **Amazon Route 53** and **Kong API Gateway**.
2. **Document Processor Service** receives document upload requests and pushes processing tasks to **Amazon MQ (RabbitMQ)**.
3. **Document Storage Service** listens for messages from **Amazon MQ**, processes documents, encrypts them, and stores them in **Amazon EFS**.
4. **Document Storage Service** stores the encrypted document’s **download URL** in **MongoDB**.
5. Once the document is stored, **Document Storage Service** sends a push notification to the mobile app via **Firebase Cloud Messaging (FCM)**.
6. Users receive a notification with a **download link** to access the stored document.
7. **AWS Backup Service** ensures periodic backups of **Amazon EFS** for disaster recovery.

---

### **✅ Key Benefits**

- **Highly Scalable**: Uses **Kubernetes auto-scaling** via **AWS EKS** and **Karpenter**.
- **Efficient Traffic Management**: Powered by **Amazon Route 53**, **ALB**, and **Istio Service Mesh**.
- **Secure & Compliant**: Encryption, **Kong API Gateway**, and **EFS encryption** ensure data security.
- **Reliable & Fault-Tolerant**: Uses **Amazon MQ** for async processing, **AWS Backup Service** for redundancy.
- **Mobile-Friendly**: Supports **Firebase Cloud Messaging** for instant user notifications.

---

#### **Dual EKS Clusters per Sprint with Istio Service Mesh**

✅ **Multi-cluster redundancy (Even/Odd Sprints)** for resilience  
✅ **Disaster recovery preparedness** (If one EKS fails, traffic shifts)  
✅ **Internal testing of new releases** by routing the Mobile & Desktop App to the passive EKS cluster

```mermaid
graph LR
  subgraph "Mobile App & Desktop App"
    subgraph "Production App"
      ProdUsers["Users (Production - Mobile & Desktop)"]
    end
    subgraph "Beta Tester App"
      BetaUsers["Users (Beta Testing - Mobile & Desktop)"]
    end
  end
  subgraph "AWS Cloud"
    subgraph "Networking & Security"
      Route53["Amazon Route 53 (Failover & Latency Routing)"]
      ALB_Odd["AWS ALB/NLB (Odd Traffic)"]
      ALB_Even["AWS ALB/NLB (Even Traffic)"]
    end
    subgraph "API Gateway Layer"
      Kong_Odd["Kong API Gateway (Odd EKS)"]
      Kong_Even["Kong API Gateway (Even EKS)"]
    end
    subgraph "EKS Deployment (Handles Live Traffic)"
      subgraph "Odd Sprint Cluster"
        EKS_Odd["AWS EKS - Odd Sprint"]
        DocService_Odd["Document Processor (Odd)"]
        StorageService_Odd["Document Storage (Odd)"]
        NotificationService_Odd["Notification Service (Odd)"]
        AmazonMQ_Odd["Amazon MQ (Odd)"]
      end
      subgraph "Even Sprint Cluster"
        EKS_Even["AWS EKS - Even Sprint"]
        DocService_Even["Document Processor (Even)"]
        StorageService_Even["Document Storage (Even)"]
        NotificationService_Even["Notification Service (Even)"]
        AmazonMQ_Even["Amazon MQ (Even)"]
      end
    end
    subgraph "Storage & Backup"
      EFS["Amazon EFS (Encrypted)"]
      MongoDB["MongoDB (Download URLs)"]
      AWSBackup["AWS Backup Service"]
    end
  end

  %% Routing logic
  ProdUsers-- "Browse & Upload Documents" --> Route53
  BetaUsers-- "Browse & Upload Documents (Beta)" --> Route53

  Route53 -->|Odd Requests| ALB_Odd
  Route53 -->|Even Requests| ALB_Even

  ALB_Odd --> Kong_Odd
  ALB_Even --> Kong_Even

  Kong_Odd --> EKS_Odd
  Kong_Even --> EKS_Even

```

### **Key Takeaways**
- **Active EKS Cluster** handles live traffic
- **Passive EKS Cluster** allows **staging & disaster recovery**
- **Failover mechanism**: If Active EKS fails, Route 53 shifts traffic
- **Testing & Validation**: Internal app users can test on Passive Cluster

---

**[Return to the index.](./Solution.md)**
## **🦜 Proposed Architecture for the Document Processor System**

| **Challenge**                  | **Proposed Solution**                                                                                                                                                                                                                                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Scalability & Performance**  | ✅ Migrate to **AWS EKS** for Kubernetes-based auto-scaling (**Karpenter**) <br> ✅ Use **RabbitMQ** for asynchronous processing |
| **Traffic Control & Routing**  | ✅ Use **Amazon Route 53** for **failover & latency-based routing** |
| **Storage & Data Security**    | ✅ Store documents in **Amazon EFS** (scalable, low latency) <br> ✅ **Encrypt documents** for security and compliance |
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
      Istio["Istio Service Mesh"]
    end
    subgraph "Compute"
      EKS["AWS EKS (Kubernetes Cluster)"]
      DocService["Document Processor Service"]
      DocStorage["Document Storage Service"]
      NotificationService["Notification Service"]
      AmazonMQ["Amazon MQ (RabbitMQ)"]
    end
    subgraph "Storage"
      EFS["Amazon EFS (Encrypted)"]
      MongoDB["MongoDB (Download URLs)"]
      AWSBackup["AWS Backup Service (EFS)"]
    end
  end

  Drivers-- "Browse and Upload Documents API" -->Route53
  Agents-- "Browse and Administer Documents API" -->Route53
  Route53 --> ALB
  ALB --> Kong
  Kong --> Istio
  Istio --> EKS
  EKS --> DocService
  DocService-- "Queue Processing" --> AmazonMQ
  DocStorage-- "Listening for Messages" --> AmazonMQ
  DocStorage-- "Process, Encrypt, Store" --> EFS
  DocStorage-- "Store Encrypted Download URL" --> MongoDB
  DocStorage-- "Send Push Notification with Download URL" --> NotificationService
  NotificationService-- "Send Push Notifications" --> Drivers
  EFS-- "Backups to" --> AWSBackup
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




#### ** Dual EKS Clusters per Sprint with Istio Service Mesh **

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
      ALB["AWS ALB/NLB"]
      Kong["Kong API Gateway (Auth, Rate Limiting)"]
    end
    subgraph "Dual EKS Deployment (Redundant & Isolated Clusters)"
      subgraph "Active Cluster (Handles Live Traffic)"
        EKS_Active["AWS EKS - Active (Even/Odd)"]
        DocService_Active["Document Processor (Active)"]
        StorageService_Active["Document Storage (Active)"]
        NotificationService_Active["Notification Service (Active)"]
        AmazonMQ_Active["Amazon MQ (Active)"]
      end
      subgraph "Passive Cluster (For Beta Testing & DR)"
        EKS_Passive["AWS EKS - Passive (Even/Odd)"]
        DocService_Passive["Document Processor (Passive)"]
        StorageService_Passive["Document Storage (Passive)"]
        NotificationService_Passive["Notification Service (Passive)"]
        AmazonMQ_Passive["Amazon MQ (Passive)"]
      end
    end
    subgraph "Storage & Backup"
      EFS["Amazon EFS (Encrypted)"]
      MongoDB["MongoDB (Download URLs)"]
      AWSBackup["AWS Backup Service"]
    end
  end

  ProdUsers-- "Browse & Upload Documents" --> Route53
  BetaUsers-- "Browse & Upload Documents (Beta)" --> Route53

  Route53 --> ALB
  ALB --> Kong
  Kong --> EKS_Active & EKS_Passive

  EKS_Active --> DocService_Active
  EKS_Passive --> DocService_Passive
  DocService_Active -->|"Send to Queue"| AmazonMQ_Active
  DocService_Passive -->|"Send to Queue"| AmazonMQ_Passive

  AmazonMQ_Active -->|"Process Storage"| StorageService_Active
  AmazonMQ_Passive -->|"Process Storage"| StorageService_Passive

  StorageService_Active -->|"Save File"| EFS
  StorageService_Passive -->|"Save File"| EFS

  StorageService_Active -->|"Store URL"| MongoDB
  StorageService_Passive -->|"Store URL"| MongoDB

  EFS-- "Backup to" --> AWSBackup

  StorageService_Active -->|"Trigger Notification"| NotificationService_Active
  StorageService_Passive -->|"Trigger Notification"| NotificationService_Passive

  NotificationService_Active -->|"Send Push Notification"| ProdUsers
  NotificationService_Passive -->|"Send Push Notification"| BetaUsers

```

### **Key Takeaways**
- **Active EKS Cluster** handles live traffic
- **Passive EKS Cluster** allows **staging & disaster recovery**
- **Failover mechanism**: If Active EKS fails, Route 53 shifts traffic
- **Testing & Validation**: Internal app users can test on Passive Cluster

---

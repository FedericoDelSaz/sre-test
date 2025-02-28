## **🔴 Key Bottlenecks & Risks**

| **Category**                 | **Bottlenecks & Risks** | **Proposed Solutions** |
|-----------------------------|------------------------|------------------------|
| **Scalability Issues**       | - Single VM for backend <br> - No autoscaling <br> - Single Load Balancer (no redundancy) | ✅ Migrate to AWS EKS for auto-scaling <br> ✅ Use multiple backend instances <br> ✅ Add redundant Load Balancers |
| **Storage & Performance**    | - NFS-based storage (high latency) <br> - RAID 1 setup (no performance boost) <br> - Synchronous processing (slow uploads) | ✅ Use **Amazon EFS** for **low-latency, shared storage** <br> ✅ Enable encryption for security <br> ✅ Introduce RabbitMQ for asynchronous processing |
| **Reliability & Fault Tolerance** | - Single point of failure (backend & storage) <br> - No offsite backups <br> - No failover or latency-based routing | ✅ Implement AWS Route 53 for traffic control <br> ✅ Set up failover routing <br> ✅ Store backups using **AWS Backup Service** for **EFS snapshots** |
| **Security & Compliance**    | - Documents stored on a single VM (risk of breaches) <br> - No API Gateway (no rate limiting or DDoS protection) <br> - No Web Application Firewall (WAF) | ✅ Use **Kong API Gateway** with **rate limiting** and **OAuth plugin** <br> ✅ Deploy **Istio Service Mesh** for traffic management & security |
| **Deployment & Release Risks** | - No Canary/Blue-Green Deployments <br> - No progressive rollouts <br> - Slow manual rollback process | ✅ Adopt Canary & Blue-Green Deployments <br> ✅ Automate rollbacks with CI/CD pipelines |
| **Mobile Notifications**     | - No notification system for mobile apps | ✅ Integrate **Firebase Cloud Messaging (FCM)** to send real-time notifications to the mobile app |

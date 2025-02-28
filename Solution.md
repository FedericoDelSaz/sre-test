# **Solution Overview**

This document provides a high-level summary of the proposed solution to enhance system reliability, scalability, and security. It links to detailed documentation covering bottlenecks and risks, the new architecture, and the action plan.

## **Key Objectives**
- **Ensure High Availability (HA)** by introducing redundancy and failover mechanisms.
- **Optimize Storage** by migrating from NFS to **AWS EFS** with lifecycle policies.
- **Secure Document Management** by storing **encrypted document URLs in MongoDB** for controlled access and downloads.
- **Containerize Applications** for easier deployment and management.
- **Migrate to EKS** in a phased approach to leverage Kubernetes scalability.
- **Implement Monitoring & Security Tools** to enhance observability and compliance.
- **Enable Real-time Notifications** by deploying a **Notification Service in EKS** integrated with **Firebase Cloud Messaging (FCM)** to send alerts to mobile apps.

## **Linked Documentation**
- **[Bottlenecks & Risks](./BottlenecksRisk.md)** – Identifies existing system limitations and potential risks.
- **[New Architecture](./NewArchitecture.md)** – Details the proposed infrastructure and technology stack.
- **[Action Plan](./ActionPlan.md)** – Outlines the step-by-step approach for implementation.

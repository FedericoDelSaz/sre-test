# SRE Architecture Test Solution

### Identify the bottlenecks and potential risks of the current approach and explain why.

## Bottlenecks & Risk

- **System is not scalable:**
    - The `Document Processor` is running standalone in a Virtual Machine, making it unable to scale efficiently to handle a large number of requests from the Mobile and Desktop Apps.

- **System is not prepared for disaster recovery:**
    - The `NFS Volume` lacks a backup plan and storage lifecycle setup.

- **System lacks High Availability:**
    - Since the system is non-scalable, a failure in the `Document Processor` could lead to a service outage. High Availability (HA) setup is not in place.

- **System is not prepared for Zero Downtime Deployment:**
    - Both the `Document Processor` and `NFS Volume` are not configured for deployment strategies like canary or blue/green. This lack of preparation could result in service downtime during deployments.

- **System is not global:**
    - As mentioned in README.md, "Our Drivers numbers are increasing quickly as the company lands in many different countries." The system needs to be globally accessible, catering to users in different countries and continents.

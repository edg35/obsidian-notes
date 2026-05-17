## Topics
- [[#AWS Well-Architected Framework]]
- [[#The Six Pillars of the Well-Architected Framework]]
- [[#AWS Well-Architected Lenses]]

---

## AWS Well-Architected Framework

The AWS Well-Architected Framework is a set of **guiding principles and best practices** that help cloud architects design, build, and maintain well-architected solutions on AWS.

Originally created by AWS Solutions Architects, it's used to:
- Evaluate architectures for potential weaknesses
- Improve decision-making and trade-off analysis
- Drive continuous improvement
- Align with AWS's best practices

The framework revolves around **six pillars**, each focusing on a key area of architectural best practices.

### 📌 Goals of the Framework
- Build secure, high-performing, resilient, and efficient infrastructure
- Avoid costly redesigns or failures
- Ensure workloads align with business goals
- Provide a **common language** between technical and business stakeholders

---

## The Six Pillars of the Well-Architected Framework

Each pillar addresses specific aspects of system design and operations:

---

### 🔐 1. Security

Focuses on protecting data, systems, and assets through risk assessment and mitigation strategies.

**Key Design Principles:**
- Implement a strong identity foundation
- Enable traceability
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest
- Prepare for security events

**AWS Services to Consider:**
- IAM, AWS Organizations, CloudTrail, Config, GuardDuty, KMS, Macie

🗂️ Suggested subnotes:
- `IAM Best Practices` (see [[Security and Compliance]])
- `Automating Security in AWS`
- `Encryption in AWS`

---

### 🧰 2. Operational Excellence

Involves running and monitoring systems to deliver business value and continuously improve processes.

**Key Design Principles:**
- Perform operations as code
- Make frequent, small, reversible changes
- Anticipate failure
- Learn from all operational events

**AWS Services to Consider:**
- CloudWatch, CloudTrail, Config, AWS Systems Manager, X-Ray

🗂️ Suggested subnotes:
- `Observability in AWS`
- `Automated Incident Response`
- `CloudWatch vs X-Ray`

---

### 💰 3. Cost Optimization

Ensures systems deliver business value at the lowest price point.

**Key Design Principles:**
- Implement cloud financial management
- Adopt a consumption model
- Measure overall efficiency
- Stop spending on undifferentiated heavy lifting
- Analyze and attribute expenditure

**AWS Services to Consider:**
- Cost Explorer, Budgets, Savings Plans, Trusted Advisor

🗂️ Suggested subnotes:
- `Rightsizing Resources`
- `Cost Explorer Tips`
- `FinOps in AWS`

---

### ⚙️ 4. Reliability

Covers the ability of a system to recover from failures and meet demands.

**Key Design Principles:**
- Automatically recover from failure
- Test recovery procedures
- Scale horizontally
- Stop guessing capacity
- Manage change automatically

**AWS Services to Consider:**
- Auto Scaling, Route 53, ELB, S3, RDS Multi-AZ, CloudFormation

🗂️ Suggested subnotes:
- `High Availability Design Patterns`
- `Elastic Load Balancing Deep Dive` (see [[AWS Computing]])
- `Fault Tolerant Architectures`

---

### 🚀 5. Performance Efficiency

Focuses on using computing resources efficiently to meet requirements as they evolve.

**Key Design Principles:**
- Democratize advanced technologies
- Go global in minutes
- Use serverless architectures
- Experiment more often
- Use mechanical sympathy (match resource to workload)

**AWS Services to Consider:**
- Lambda, DynamoDB, API Gateway, Aurora Serverless, CloudFront

🗂️ Suggested subnotes:
- `Serverless Performance Tips`
- `Caching Strategies`
- `DynamoDB vs RDS Performance` (see [[Storage]])

---

### 🛡️ 6. Sustainability

The newest pillar, focused on minimizing environmental impacts of workloads.

**Key Design Principles:**
- Understand impact
- Establish sustainability goals
- Maximize utilization
- Optimize software and hardware
- Reduce downstream impact

**AWS Services to Consider:**
- Graviton instances, S3 Intelligent-Tiering, AWS Compute Optimizer, AWS Sustainability Tool

🗂️ Suggested subnotes:
- `Green Cloud Computing`
- `Graviton Performance vs Cost`
- `Sustainability Metrics in AWS`

---

## AWS Well-Architected Lenses

Lenses are **specialized adaptations** of the Well-Architected Framework tailored to specific domains or use cases.

### 🌐 Examples of AWS Lenses:
- **Serverless Lens** – Optimizes event-driven and serverless workloads
- **SaaS Lens** – Guidance for building scalable and secure SaaS applications
- **Machine Learning Lens** – Focused on the ML lifecycle from data ingestion to model deployment
- **High Performance Computing (HPC) Lens**
- **IoT Lens**
- **Financial Services Lens**

Each lens reinterprets the six pillars for the context of that domain and offers:
- Common architectural patterns
- Anti-patterns to avoid
- Domain-specific best practices
- Checklists and design questions

🗂️ Suggested subnotes:
- `Serverless Well-Architected Review`
- `ML Lens for Data Scientists`
- `IoT Architecture Best Practices`

🔁 Related notes:
- `AWS Architecture Diagrams`
- `Solution Architecture Examples`
- `Well-Architected Review Process`

---
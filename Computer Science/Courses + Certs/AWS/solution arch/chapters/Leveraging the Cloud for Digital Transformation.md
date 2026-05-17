## Topics
- [[#☁️ Cloud Computing Models]] (PaaS, IaaS, SaaS)
- [[#🚚 Cloud Migration Strategy]]
- [[#📝 Implementing a Digital Transformation Strategy]]
- [[#🖼️ AWS Cloud Adoption Framework (AWS CAF)]]
- [[#🏛️ Architectures to provide high availability, reliability, and scalability]]

## ☁️ Cloud Computing Models

> [!NOTE] Definition
> Cloud computing models define **how services are delivered** and how much **responsibility and control** the consumer has versus the cloud provider. Understanding these is foundational for AWS Solution Architecture

### 🔧 IaaS – Infrastructure as a Service

> [!NOTE] Definition
> IaaS provides **on-demand access to raw computing infrastructure** like virtual machines, storage, and networking. The cloud provider manages the underlying physical infrastructure, while you control everything above the OS layer.
#### **User responsibilities**
- Install and manage operating systems
- Deploy runtime environments, middleware, and applications
- Handle data, patches, monitoring, and scaling
#### **Cloud provider responsibilities**
- Physical servers, virtualization, networking, and storage
- Data center maintenance
- Hypervisors and host OS
#### **Examples**
- Amazon EC2
- Azure Virtual Machines
- Google Compute Engine
- DigitalOcean Droplets
#### **Use cases**
- Hosting custom applications
- Migrating legacy apps ("lift-and-shift")
- Testing and development environments
- High-performance computing
#### **Benefits**
- Maximum flexibility and control
- Scalable infrastructure
- Pay-as-you-go model (see [[Benefits of AWS]])
#### **Tradeoffs**
- More operational overhead
- Requires expertise in system administration
- Responsibility for patching and securing the OS

### 🧱 PaaS – Platform as a Service

> [!NOTE] Definition
> PaaS provides a **ready-to-use platform** for application development and deployment. You don’t manage servers or the operating system; you just focus on writing and deploying code.

#### **User responsibilities**
- Write application code
- Manage data and business logic
#### **Cloud provider responsibilities**
- Infrastructure, OS, runtime, middleware
- Load balancing, scaling, backups
- Monitoring and platform updates
#### **Examples**
- AWS Elastic Beanstalk
- Google App Engine
- Azure App Services
- Heroku
#### **Use cases**
- Rapid application development
- Microservices-based apps
- Continuous Integration/Continuous Delivery (CI/CD) pipelines
- Web and mobile app backends
#### **Benefits**
- Faster development cycles
- Reduced operational overhead
- Built-in scalability and high availability
#### **Tradeoffs**
- Less control over environment and configuration
- Possible vendor lock-in
- Limited customization compared to IaaS

### 📦 [[SaaS]] – Software as a Service

> [!NOTE] Definition
> SaaS delivers **fully functional software** applications over the internet. Users access the software via a web browser or API. The provider handles all aspects of the infrastructure, platform, and application.

#### **User responsibilities**
- Use the software as-is
- Manage user accounts and permissions
#### **Cloud provider responsibilities**
- Full stack: infrastructure, platform, and application
- Patching, maintenance, uptime
- Security and compliance
#### **Examples**
- Gmail, Google Workspace
- Salesforce
- Dropbox
- Microsoft 365
- Zoom
#### **Use cases**
- Collaboration tools (email, document sharing)
- CRM and ERP solutions
- Analytics dashboards
- E-learning and video conferencing
#### **Benefits**
- Zero maintenance
- Quick onboarding
- Scales with usage
- Accessible from anywhere
#### **Tradeoffs**
- Minimal customization
- Data residency concerns
- Security is mostly out of your control

### Comparing Models
- See [[Cloud Strategy Decision Matrix]]
- See [[Cost Comparison Between Models]]

| Model | Managed by Customer     | Managed by Provider         | Key Examples              | Best For             |
| ----- | ----------------------- | --------------------------- | ------------------------- | -------------------- |
| IaaS  | Apps, Data, OS, Runtime | Servers, Virtualization     | EC2, GCE                  | Custom workloads     |
| PaaS  | Apps, Data              | Runtime, OS, Infrastructure | Elastic Beanstalk, Heroku | Fast app development |
| SaaS  | Just use the app        | Everything else             | Gmail, Salesforce         | End-user tools       |

## 🚚 Cloud Migration Strategy

> [!NOTE] Definition
> Cloud migration is the process of moving digital assets (applications, data, workloads, services) from on-premises or other clouds to AWS. A successful migration strategy involves **planning, execution, and ongoing optimization**.

### Links
- AWS Well-Architected Framework (see [[Understanding the AWS Well-Architected Framework and Getting Certified]])
- Security and Compliance in AWS (see [[Networking in AWS]], [[Security and Compliance]])
- Multi-Account Strategy (see [[Security and Compliance]])

### 🛣️ The 3 Phases of Migration
#### 1. **Assess**
- Understand current workloads, dependencies, and requirements.
- Evaluate **readiness** for cloud adoption.
- Perform a **TCO analysis** and identify business drivers (e.g., agility, cost savings).
- Classify applications (mission-critical, legacy, redundant).

📌 Tools:
- AWS Migration Evaluator (formerly TSO Logic)
- AWS Application Discovery Service
- AWS Well-Architected Tool (see [[Understanding the AWS Well-Architected Framework and Getting Certified]])
#### 2. **Mobilize**
- Build the foundation (landing zone, security, governance).
- Create a **migration plan** with prioritized workloads.
- Address **gaps** found during the assessment (e.g., skillsets, tooling).
- Establish baseline **cloud architecture** and pipelines.

📌 Tools:
- AWS Landing Zone
- AWS Control Tower
- AWS Migration Hub
#### 3. **Migrate & Modernize**
- Execute the migration based on selected patterns (see 7 R’s below).
- Validate, test, and optimize.
- Consider **modernization** (refactoring, rearchitecting).
- Set up **observability, autoscaling, and cost monitoring**.

📌 Tools:
- AWS Server Migration Service (SMS)
- AWS Database Migration Service (DMS)
- CloudEndure Migration (now part of AWS Application Migration Service)

### 🔁 The 7 R’s of Migration Patterns

![[Screenshot 2025-04-21 at 9.14.19 PM 3.png ]]

|Strategy|Description|When to Use|
|---|---|---|
|**1. Rehost** (Lift and Shift)|Move as-is to the cloud. No code changes.|Fast migrations; legacy apps; time/budget constraints|
|**2. Replatform** (Lift, Tinker, and Shift)|Minor changes (e.g., database engine swap, OS tuning)|When app benefits from cloud optimizations but full refactor is unnecessary|
|**3. Repurchase**|Switch to SaaS or commercial solution|Legacy CRM → Salesforce; proprietary tools → SaaS replacements|
|**4. Refactor / Re-architect**|Major code changes to take advantage of cloud-native features|Monolith → microservices; legacy → serverless|
|**5. Retire**|Decommission unused or obsolete apps|Shadow IT, redundant systems, or end-of-life software|
|**6. Retain**|Keep on-premises for now|Regulatory, compliance, or technical limitations|
|**7. Relocate**|Move entire environments with minimal changes (e.g., VMware Cloud on AWS)|Data center migration with tight deadlines or infrastructure coupling|
## 📝 Implementing a Digital Transformation Strategy

 **Two Critical Decisions Need to be Made:**
 - Minimum amount of tasks required to achieve migration or take opportunity to to enhance, change, and optimize services?
 - Should migration be purely technological or should you take the opportunity to reform current business processes?

### ❓ What Exactly is Digital Transformation?


> [!NOTE] Definition
> Involves using the cloud or other advanced technology to create new or change existing business flows
> 
> It often involves changing a company culture to adapt to new business type. The primary goal is to enhance customer experience and adapt to ever changing business demand

#### **Primarily seen as the Opportunity to Reconsider the Following:**
- current structures of teams and departments
- current business flows
- the way new functionality is developed

### 🚘 Digital Transformation Drivers

- Disrupt yourself or be disrupted by the competition
- Inability to adapt forced many of Amazons competitors to file for bankruptcy

### 🗒️ Digital Transformation Tips

#### Tip #1 - Ask the right questions
- The main question <span style="color:rgb(0, 176, 80)">How can we do what we are doing faster and better?</span> can be broken down further: 
	- How can we change what we are doing to serve our customer better?
	- Can we eliminate certain lines of business, departments, or processes?
	- What business outcomes do we want to achieve when interfacing with our customers?
	- What happens if we do not do anything?
	- What are our competitors doing?
#### Tip #2 - Get Leadership Buy-in
- In order for full systemic changes to take place, leadership needs to be fully bought in
- While [[Proof of Concept]] (POC) can still be performed at a department level, but will lead to other apartments handling business processes differently until adopted
#### Tip #3 - Deadline objectives
- While agile development will bring about changes in direction as new requirements are discovered, the overall objective of a digital transformation should be crystal clear at all times
#### Tip #4 - Apply agile methodology to your digital transformation
- Need healthy balance between delivering on objects and implementations of new requirements
- Agile increases ROI by capitalizing of new features ASAP
- Prioritizes hitting singles over home runs
	- This will better validate approach to leadership
- Pick workloads that you can migrate quickly and do those first
#### Tip #5 - Encourage risk taking
- Also known as failing fast
- Learn to reuse material from past failures for future success
#### Tip #6 - One-way door vs. two-way door decisions
- One-way door decision:
	- A decision where once you start, it will be too costly to roll back
- Two-way door decision:
	- A decision that you can go back on easily for little to no extra cost
	- Clear exit strategy that will not impact end user experience
- Examples:
	-  One-way: Migrating your ecommerce platform to the cloud
	-  Two-way: Migrating your HR payroll to the cloud, and buy ADP if migration takes too long or costs too much

#### Tip #7 - Clear delineation of roles and responsibilities 
- Fully outline roles and responsibilities in order to maximize efficiency.
- Prioritize experts in smaller fields

### 📉 Digital Transformation Pitfalls
- Many ways to fail and fewer ways to succeed, but there are common failure patterns to watch out for
#### Lack of commitment from the C-suite
- Even if the CEO wants to transform the business, they goals vs action plane my not be totally in sync
#### Not having the right team in place
- It may be worth it to use people who have had experience with a similar type of migration
- Engaging with companies that specialize in cloud migrations may be beneficial, but often are expensive
#### Internal Resistance
- Watch out for turnover in the team as it can cause friction
- Interpersonal relationships may also affect progress
- Certain roles, like Infrastructure or system admins tend to resist cloud adoption, as it generally reduces or changes their responsibilities
#### Going too fast
- make sure you prove concepts at smaller scales before you move forward
	- this allows for the identification of mistakes before taking a solution enterprise wide
- Do small PoC projects first, (move a few databases to the cloud before moving all 100)
#### Going too slow
- After successful PoC, don't keep moving department at a time. Know when the right time is to fully scale up
- After the sustainable template is setup, you can rollout changes across the board
#### Outdated Rules and Regulations
- rules and regulations that are external to the company also often play a factor
	- government regulations preventing new technology from making old processes more efficient

## 🖼️ AWS Cloud Adoption Framework (AWS CAF)
- As discussed, cloud adoption is not an exact science and as a result, some companies my struggle to fully realize the benefits of moving workloads to the cloud. Thus the AWS CAF is used to further validate the expected benefits of a migration
- The CAF helps to identify business outcomes such as risk, performance, revenue, and operational productivity![AWS Cloud Adoption Framework (CAF) 3.0 is Now Available | AWS News Blog](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2021/11/04/caf_v3_3.png)
### 🪜 Four phases 
#### Envision
- Understand business transformation opportunities and define quantified business outcomes to drive value
#### Align
- Identify gaps and dependencies in organization to create a plan for cloud readiness
#### Launch
- Build proof of concept and deliver pilots to define future direction toward production
#### Scale
- scale pilots to take into production

### 🫥 Four Transformation Domains
#### Technology transformation
- using cloud migration to modernized approach in the cloud
#### Process transformation
- a data and analytics approach using cloud technology
#### Organizational transformation
- building an efficient operation model in the cloud
#### Product transformation
- building cloud focused business and revenue models

## 🏛️ Architectures to provide high availability, reliability, and scalability

> [!NOTE] **Redundancy**: 
> concept of copying data across resources to increase reliability and availability. While relatively easy with systems, it is much hard to implement in practice with databases.
> 
> With regard to databases, there is a good and bad version of redundancy
> 
> **Good**: data replication across multiple environments 
> **Bad**: denormalized database tables or manual copies of files


### 📋 Types of Application Architectures
#### Active Architecture
- single point of failure, aka functional architecture
- if your hard drive fails all data is lost

![[Screenshot 2025-05-02 at 4.44.00 PM.png]]

#### Active/passive architecture
- implements a simple backup
- lets one node (active node) handle all reads and writes and syncs state with passive node

![[Screenshot 2025-05-02 at 4.47.33 PM.png]]

- this not only provides a fresh copy of critical data, but also allows traffic to be redirected to the passive node in case of failure

![[Screenshot 2025-05-02 at 4.49.40 PM.png]]

- in the first generations of active/passive architecture, **synchronous transactions** were used which means the passive node had to acknowledge the writes before they would take affect. This would cause a bottleneck in the system if the passive node ever went down
- As a solution, we moved to **asychronous transactions**  which is a store and forward method to back up data. This method stored data in the primary node first before and then sends a request to the passive node separately while continuing usually work

**Drawbacks**:
- system will still fail if both nodes go down
- data not replicated to the passive node before an active node failure will be lost
- since passive node is used as a backup, the throughput is limited to the capacity of the active node, and the passive nodes throughput is wasted
#### Active/active architecture
- created to handle the "always on" demands of users as systems grew
- replicates workloads across multiple separate physical locations and uses load balancers to distribute traffic
- This feature is implemented easily in aws and can either operated with fully active/active or replicated at a lower cost using warm stand-by.
- Expensive, but basically 100% fault tolerant
#### Sharding Architecture
- architecture where there are multiple nodes that all participate in handling traffic
- this method parcels out work based on a predefined scheme
	- use a primary key to determine which shard to send work to, just make sure primary keys are balanced so one shard does not receive more work than the others
- This provides much more overhead and is complex to implement
### 🌋 Chaos engineering

> [!NOTE] Chaos Engineering
> a methodology devoted to building resilient systems by purposely trying to break them and expose their weaknesses

- any system needs a well thought out plan in order to deal with failure
- also includes a disaster recovery plan in case of failure

> [!NOTE] Recovery Point Objective (RPO)
> contents


> [!NOTE] Recovery Time Objective (RTO)
> Contents

## Learning Objectives
- Describe the benefits of Amazon EC2 at a basic level.
- Identify the different Amazon EC2 instance types.
- Differentiate between the various billing options for Amazon EC2.
- Summarize the benefits of Amazon EC2 Auto Scaling.
- Summarize the benefits of Elastic Load Balancing.
- Give an example of the uses for Elastic Load Balancing.
- Summarize the differences between Amazon Simple Notification Service (Amazon SNS) and Amazon Simple Queue Service (Amazon SQS).
- Summarize additional AWS compute options.

## Amazon Elastic Computer Cloud (EC2)

> [!NOTE] Info
> [Amazon Elastic Compute Cloud (Amazon EC2)(opens in a new tab)](https://aws.amazon.com/ec2/) provides secure, resizable compute capacity in the cloud as Amazon EC2 instances.


**How it works**:

1. First, you launch an instance. Begin by selecting a template with basic configurations for your instance. These configurations include the operating system, application server, or applications. You also select the instance type, which is the specific hardware configuration of your instance. As you are preparing to launch an instance, you specify security settings to control the network traffic that can flow into and out of your instance. Later in this course, we will explore Amazon EC2 security features in greater detail.
2. Next, connect to the instance. You can connect to the instance in several ways. Your programs and applications have multiple different methods to connect directly to the instance and exchange data. Users can also connect to the instance by logging in and accessing the computer desktop.
3. After you have connected to the instance, you can begin using it. You can run commands to install software, add storage, copy and organize files, and more.

## Amazon EC2 Instance Types


> [!NOTE] Info
> [Amazon EC2 instance types(opens in a new tab)](https://aws.amazon.com/ec2/instance-types/) are optimized for different tasks. When selecting an instance type, consider the specific needs of your workloads and applications. This might include requirements for compute, memory, or storage capabilities.

### General Purpose Instances

**General purpose instances** provide a balance of compute, memory, and networking resources. You can use them for a variety of workloads, such as:

- application servers
- gaming servers
- backend servers for enterprise applications
- small and medium databases

Suppose that you have an application in which the resource needs for compute, memory, and networking are roughly equivalent. You might consider running it on a general purpose instance because the application does not require optimization in any single resource area.

### Compute Optimized Instance

**Compute optimized instances** are ideal for compute-bound applications that benefit from high-performance processors. Like general purpose instances, you can use compute optimized instances for workloads such as web, application, and gaming servers.

However, the difference is compute optimized applications are ideal for high-performance web servers, compute-intensive applications servers, and dedicated gaming servers. You can also use compute optimized instances for batch processing workloads that require processing many transactions in a single group.

### Memory Optimized Instance

**Memory optimized instances** are designed to deliver fast performance for workloads that process large datasets in memory. In computing, memory is a temporary storage area. It holds all the data and instructions that a central processing unit (CPU) needs to be able to complete actions. Before a computer program or application is able to run, it is loaded from storage into memory. This preloading process gives the CPU direct access to the computer program.

Suppose that you have a workload that requires large amounts of data to be preloaded before running an application. This scenario might be a high-performance database or a workload that involves performing real-time processing of a large amount of unstructured data. In these types of use cases, consider using a memory optimized instance. Memory optimized instances enable you to run workloads with high memory needs and receive great performance.

### Accelerated Computing Instance

**Accelerated computing instances** use hardware accelerators, or coprocessors, to perform some functions more efficiently than is possible in software running on CPUs. Examples of these functions include floating-point number calculations, graphics processing, and data pattern matching.

In computing, a hardware accelerator is a component that can expedite data processing. Accelerated computing instances are ideal for workloads such as graphics applications, game streaming, and application streaming.

### Storage Optimized Instance

**Storage optimized instances** are designed for workloads that require high, sequential read and write access to large datasets on local storage. Examples of workloads suitable for storage optimized instances include distributed file systems, data warehousing applications, and high-frequency online transaction processing (OLTP) systems.

## Amazon EC2 Pricing

### On Demand

**On-Demand Instances** are ideal for short-term, irregular workloads that cannot be interrupted. No upfront costs or minimum contracts apply. The instances run continuously until you stop them, and you pay for only the compute time you use.  
  
Sample use cases for On-Demand Instances include developing and testing applications and running applications that have unpredictable usage patterns. On-Demand Instances are not recommended for workloads that last a year or longer because these workloads can experience greater cost savings using Reserved Instances.

### Reserved Instances

**Reserved Instances** are a billing discount applied to the use of On-Demand Instances in your account. There are two available types of Reserved Instances:

- Standard Reserved Instances
- Convertible Reserved Instances

You can purchase Standard Reserved and Convertible Reserved Instances for a 1-year or 3-year term. You realize greater cost savings with the 3-year option. 

**Standard Reserved Instances**: This option is a good fit if you know the EC2 instance type and size you need for your steady-state applications and in which AWS Region you plan to run them. Reserved Instances require you to state the following qualifications:

- **Instance type and size:** For example, m5.xlarge
- **Platform description (operating system):** For example, Microsoft Windows Server or Red Hat Enterprise Linux
- **Tenancy:** Default tenancy or dedicated tenancy

You have the option to specify an Availability Zone for your EC2 Reserved Instances. If you make this specification, you get EC2 capacity reservation. This ensures that your desired amount of EC2 instances will be available when you need them. 

**Convertible Reserved Instances**: If you need to run your EC2 instances in different Availability Zones or different instance types, then Convertible Reserved Instances might be right for you. Note: You trade in a deeper discount when you require flexibility to run your EC2 instances.

At the end of a Reserved Instance term, you can continue using the Amazon EC2 instance without interruption. However, you are charged On-Demand rates until you do one of the following:

- Terminate the instance.
- Purchase a new Reserved Instance that matches the instance attributes (instance family and size, Region, platform, and tenancy).

### EC2  Saving Plans

AWS offers Savings Plans for a few compute services, including Amazon EC2. **EC2 Instance Savings Plans** reduce your EC2 instance costs when you make an hourly spend commitment to an instance family and Region for a 1-year or 3-year term. This term commitment results in savings of up to 72 percent compared to On-Demand rates. Any usage up to the commitment is charged at the discounted Savings Plans rate (for example, $10 per hour). Any usage beyond the commitment is charged at regular On-Demand rates.

The EC2 Instance Savings Plans are a good option if you need flexibility in your Amazon EC2 usage over the duration of the commitment term. You have the benefit of saving costs on running any EC2 instance within an EC2 instance family in a chosen Region (for example, M5 usage in N. Virginia) regardless of Availability Zone, instance size, OS, or tenancy. The savings with EC2 Instance Savings Plans are similar to the savings provided by Standard Reserved Instances.

Unlike Reserved Instances, however, you don't need to specify up front what EC2 instance type and size (for example, m5.xlarge), OS, and tenancy to get a discount. Further, you don't need to commit to a certain number of EC2 instances over a 1-year or 3-year term. Additionally, the EC2 Instance Savings Plans don't include an EC2 capacity reservation option.

### Spot Instance

**Spot Instances** are ideal for workloads with flexible start and end times, or that can withstand interruptions. Spot Instances use unused Amazon EC2 computing capacity and offer you cost savings at up to 90% off of On-Demand prices.

### Dedicated Host
**Dedicated Hosts** are physical servers with Amazon EC2 instance capacity that is fully dedicated to your use. 

You can use your existing per-socket, per-core, or per-VM software licenses to help maintain license compliance. You can purchase On-Demand Dedicated Hosts and Dedicated Hosts Reservations. Of all the Amazon EC2 options that were covered, Dedicated Hosts are the most expensive.

## Scaling Amazon EC2

### Scalability

- **Scalability** involves beginning with only the resources you need and designing your architecture to automatically respond to changing demand by scaling out or in. 
- **Amazon EC2 Auto Scaling** provides automatic scaling for EC2 instances. There are two types of scaling which can be used separately or together
	- _Dynamic scaling_ responds to changing demand
	- _Predictive scaling_ automatically schedules the right number of Amazon EC2 instances based on predicted demand
- You can also scale instances vertically or horizontally as well
	- **Vertical**: increase compute power of EC2 instance
	- **Horizontal**: increase number of EC2 instances
- Auto Scaling Group:
	- **minimum capacity** is the number of Amazon EC2 instances that launch immediately after you have created the Auto Scaling group.
	- **desired capacity** at two Amazon EC2 instances even though your application needs a minimum of a single Amazon EC2 instance to run
	- **maximum capacity** maximum of four Amazon EC2 instances

## Elastic Load Balancing

> [!NOTE] Elastic Load Balancing
>  the AWS service that automatically distributes incoming application traffic across multiple resources, such as Amazon EC2 instances.

![[Pasted image 20250103122303.png]]
### Scalability

- **Low-demand period**: If only a few registers are open, this matches the demand of customers who need service
- **High-demand period**: as the number of customers increases, the coffee shop opens more registers to accommodate them

## Messaging and Queuing


> [!NOTE] Monolithic Application
> Suppose that you have an application with tightly coupled components. These components might include databases, servers, the user interface, business logic, and so on.

![[Pasted image 20250103123405.png]]

> [!NOTE] Microservice
> application components are loosely coupled. In this case, if a single component fails, the other components continue to work because they are communicating with each other. The loose coupling prevents the entire application from failing
> 

### Amazon Simple Notification Service (SNS)
- **Amazon Simple Notification Service (Amazon SNS)** is a publish/subscribe service. Using Amazon SNS topics, a publisher publishes messages to subscribers. This is similar to the coffee shop; the cashier provides coffee orders to the barista who makes the drinks.
- In Amazon SNS, subscribers can be web servers, email addresses, AWS Lambda functions, or several other options.

## Amazon Simple Queue Service (SQS)
- **Amazon Simple Queue Service (Amazon SQS)** is a message queuing service.
- Using Amazon SQS, you can send, store, and receive messages between software components, without losing messages or requiring other services to be available. In Amazon SQS, an application sends messages into a queue. A user or service retrieves a message from the queue, processes it, and then deletes it from the queue.

## Additional Services

### Serverless Computing

- [**AWS Lambda**(opens in a new tab)](https://aws.amazon.com/lambda) is a service that lets you run code without needing to provision or manage servers. While using AWS Lambda, you pay only for the compute time that you consume.

### Containers

- **Containers** provide you with a standard way to package your application's code and dependencies into a single object. You can also use containers for processes and workflows in which there are essential requirements for security, reliability, and scalability.

#### Amazon Elastic Container Service (ECS)
- [**Amazon Elastic Container Service (Amazon ECS)**(opens in a new tab)](https://aws.amazon.com/ecs/) is a highly scalable, high-performance container management system that enables you to run and scale containerized applications on AWS.

#### Amazon Elastic Kubernetes Service (EKS)
- [**Amazon Elastic Kubernetes Service (Amazon EKS)**(opens in a new tab)](https://aws.amazon.com/eks/) is a fully managed service that you can use to run Kubernetes on AWS.

#### AWS Fargate
- [**AWS Fargate**(opens in a new tab)](https://aws.amazon.com/fargate/) is a serverless compute engine for containers. It works with both Amazon ECS and Amazon EKS.
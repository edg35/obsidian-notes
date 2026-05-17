## Learning Objectives

- Describe the basic concepts of networking.
- Describe the difference between public and private networking resources. 
- Explain a virtual private gateway using a real life scenario. 
- Explain a virtual private network (VPN) using a real life scenario.
- Describe the benefit of AWS Direct Connect. 
- Describe the benefit of hybrid deployments. 
- Describe the layers of security used in an IT strategy.
- Describe the services customers use to interact with the AWS global network.

## Amazon Virtual Private Cloud (VPC)

- Amazon VPC enables you to provision an isolated section of the AWS Cloud. In this isolated section, you can launch resources in a virtual network that you define. Within a virtual private cloud (VPC), you can organize your resources into subnets. A **subnet** is a section of a VPC that can contain resources such as Amazon EC2 instances.

### Internet Gateway

To allow public traffic from the internet to access your VPC, you attach an **internet gateway** to the VPC.

![[Pasted image 20250106130140.png]]

- An internet gateway is a connection between a VPC and the internet. You can think of an internet gateway as being similar to a doorway that customers use to enter the coffee shop. Without an internet gateway, no one can access the resources within your VPC.

### Virtual Private Gateway

To access private resources in a VPC, you can use a **virtual private gateway**.

![[Pasted image 20250106130250.png]]
- A virtual private gateway enables you to establish a virtual private network (VPN) connection between your VPC and a private network, such as an on-premises data center or internal corporate network. A virtual private gateway allows traffic into the VPC only if it is coming from an approved network.

### AWS Direct Connect

[**AWS Direct Connect**(opens in a new tab)](https://aws.amazon.com/directconnect/) is a service that lets you to establish a dedicated private connection between your data center and a VPC.

![[Pasted image 20250106130603.png]]

- The private connection that AWS Direct Connect provides helps you to reduce network costs and increase the amount of bandwidth that can travel through your network.

## Subnets and Network Access Control Lists

### Subnets

- A subnet is a section of a VPC in which you can group resources based on security or operational needs. Subnets can be public or private.

![[Pasted image 20250113122001.png]]

- **Public subnets** contain resources that need to be accessible by the public, such as an online store’s website.

- **Private subnets** contain resources that should be accessible only through your private network, such as a database that contains customers’ personal information and order histories.


> [!NOTE] Example
> In a VPC, subnets can communicate with each other. For example, you might have an application that involves Amazon EC2 instances in a public subnet communicating with databases that are located in a private subnet.


### Network Traffic in a VPC

- When a customer requests data from an application hosted in the AWS Cloud, this request is sent as a packet. A **packet** is a unit of data sent over the internet or a network.
- It enters into a VPC through an internet gateway. Before a packet can enter into a subnet or exit from a subnet, it checks for permissions. These permissions indicate who sent the packet and how the packet is trying to communicate with the resources in a subnet.
- The VPC component that checks packet permissions for subnets is a [**network access control list (ACL)**(opens in a new tab)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html).

### Network ACLs

- A network ACL is a virtual firewall that controls inbound and outbound traffic at the subnet level.

Each AWS account includes a default network ACL. When configuring your VPC, you can use your account’s default network ACL or create custom network ACLs. 

By default, your account’s default network ACL allows all inbound and outbound traffic, but you can modify it by adding your own rules. For custom network ACLs, all inbound and outbound traffic is denied until you add rules to specify which traffic to allow. Additionally, all network ACLs have an explicit deny rule. This rule ensures that if a packet doesn’t match any of the other rules on the list, the packet is denied.
#### Stateless Packet Filtering

- Network ACLs perform **stateless** packet filtering. They remember nothing and check packets that cross the subnet border each way: inbound and outbound.

![[Pasted image 20250113122636.png]]

After a packet has entered a subnet, it must have its permissions evaluated for resources within the subnet, such as Amazon EC2 instances. 

The VPC component that checks packet permissions for an Amazon EC2 instance is a [**security group**(opens in a new tab)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html).

### Security Group

- A security group is a virtual firewall that controls inbound and outbound traffic for an Amazon EC2 instance.

By default, a security group denies all inbound traffic and allows all outbound traffic. You can add custom rules to configure which traffic should be allowed; any other traffic would then be denied

#### Stateful Packet Filtering

- Security groups perform **stateful** packet filtering. They remember previous decisions made for incoming packets.

When a packet response for that request returns to the instance, the security group remembers your previous request. The security group allows the response to proceed, regardless of inbound security group rules.

![[Pasted image 20250113122954.png]]

With both network ACLs and security groups, you can configure custom rules for the traffic in your VPC. As you continue to learn more about AWS security and networking, make sure to understand the differences between network ACLs and security groups.

![[Pasted image 20250113123055.png]]

## Global Networking

### DNS

- Suppose that AnyCompany has a website hosted in the AWS Cloud. Customers enter the web address into their browser, and they are able to access the website. This happens because of **Domain Name System (DNS)** resolution. DNS resolution involves a customer DNS resolver communicating with a company DNS server.

![[Pasted image 20250113125033.png]]


### Amazon Route 53

- [**Amazon Route 53**(opens in a new tab)](https://aws.amazon.com/route53) is a DNS web service. It gives developers and businesses a reliable way to route end users to internet applications hosted in AWS.

- Amazon Route 53 connects user requests to infrastructure running in AWS (such as Amazon EC2 instances and load balancers). It can route users to infrastructure outside of AWS.

![[Pasted image 20250113125227.png]]


## Amazon Database Migration Service (DMS)

- [**AWS Database Migration Service (AWS DMS)**(opens in a new tab)](https://aws.amazon.com/dms/) enables you to migrate relational databases, nonrelational databases, and other types of data stores.
- 
With AWS DMS, you move data between a source database and a target database. [The source and target databases(opens in a new tab)](https://aws.amazon.com/dms/resources) can be of the same type or different types. During the migration, your source database remains operational, reducing downtime for any applications that rely on the database.

### Other Use Cases

#### Development and test database migrations
- Enabling developers to test applications against production data without affecting production users

#### Database consolidation
- Combining several databases into a single database

#### Continuous replication
- Sending ongoing copies of your data to other target sources instead of doing a one-time migration

## Additional Database Services

- [**Amazon DocumentDB**(opens in a new tab)](https://aws.amazon.com/documentdb) is a document database service that supports MongoDB workloads. (MongoDB is a document database program.)
- [**Amazon Neptune**(opens in a new tab)](https://aws.amazon.com/neptune) is a graph database service. You can use Amazon Neptune to build and run applications that work with highly connected datasets, such as recommendation engines, fraud detection, and knowledge graphs.
- [**Amazon Quantum Ledger Database (Amazon QLDB)**(opens in a new tab)](https://aws.amazon.com/qldb) is a ledger database service. You can use Amazon QLDB to review a complete history of all the changes that have been made to your application data.
- [**Amazon Managed Blockchain**(opens in a new tab)](https://aws.amazon.com/managed-blockchain) is a service that you can use to create and manage blockchain networks with open-source frameworks. Blockchain is a distributed ledger system that lets multiple parties run transactions and share data without a central authority.
- [**Amazon ElastiCache**(opens in a new tab)](https://aws.amazon.com/elasticache) is a service that adds caching layers on top of your databases to help improve the read times of common requests. It supports two types of data stores: Redis and Memcached.
- [**Amazon DynamoDB Accelerator (DAX)**(opens in a new tab)](https://aws.amazon.com/dynamodb/dax/) is an in-memory cache for DynamoDB. It helps improve response times from single-digit milliseconds to microseconds.
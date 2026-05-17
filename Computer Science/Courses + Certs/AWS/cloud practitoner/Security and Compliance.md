
## Learning objectives

- Explain the benefits of the shared responsibility model.
- Describe multi-factor authentication (MFA).
- Differentiate between the AWS Identity and Access Management (IAM) security levels.
- Explain the main benefits of AWS Organizations.
- Describe security policies at a basic level.
- Summarize the benefits of compliance with AWS.
- Explain additional AWS security services at a basic level.

## Shared Responsibility Model
- AWS is responsible for security of the cloud
-  you are responsible for security in the cloud
-  Customer: your content and access rights
-  AWS:
	-  physical data center security
	-  Hardware and software
	-  network infrastructure, virtualization

## User Permissions
AWS IAM: manage access to AWS services and resources
-  IAM users, groups, roles
-  policies
-  MFA

AWS rost user:
- First identity when creating account,
-  can do anything in AWS
-  do not use for everyday tasks, instead make an IAM user

IAM user:
-  Identity you create in Aros with certain permissions (policies)

IAM policies:
-  documents that allow or deny permissions in AWS

![[Screenshot 2025-04-21 at 9.53.15 PM.png]]

IAM Groups:
- collection of IAM users
- When you assign a policy to a group, everyone has those permissions

IAM Roles:
- Identity where you give up access of on work station and gain access to another
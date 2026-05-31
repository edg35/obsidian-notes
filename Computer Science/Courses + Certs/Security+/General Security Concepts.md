---
Chapter: 1
tags:
---
# 1.1
## Security Controls
- security risks are out there and there are many types to consider
- assets are also varied
	- data, physical property, computer systems
- prevent security events, minimize the impact, and limit the damage
	- security controls

### Control Categories
**4 main control categories**
#### Technical Controls
- Controls implemented using systems
- operating system controls
- firewalls, antivirus

#### Managerial Controls
- administrative controls associated with security design and implementation
- security polices, standard operating procedures

#### Operational Controls
- controls implemented by people instead of systems
- security guards, awareness programs

#### Physical Controls
- Limit access
- guard shack
- fences, locks
- badge readers

### Control Types and Categories
**Control Type examples**

| Categories  | Preventative       | Deterrent      | Detective        | Corrective                   | Compensating                    | Directive                |
| ----------- | ------------------ | -------------- | ---------------- | ---------------------------- | ------------------------------- | ------------------------ |
| Technical   | Firewall           | Splash screen  | System logs      | Bakcup recovery              | Block instead of path           | file storage polices     |
| Managerial  | On-Boarding Policy | Demotion       | Login reports    | Policies for reporting issue | Separation of duties            | compliance policies      |
| Operational | Gaurd Shack        | Reception desk | Property Patrols | Contact authorities          | Require multiple security staff | security policy training |
| Physical    | Door Lock          | Warning sgns   | Motion detectors | Fire extinguisher            | Power generator                 | signs                    |

#### Preventive Control Types
- preventive
	- block access to a resource
	- you shall not pass
- prevent access
	- firewall rules
	- follow security policy
	- guard shack checks all identification
	- enable door locks

#### Deterrent Control Types
- deterrent
	- discourage an intrusion attempt
	- does not directly prevent access
- make and attacker think twice
	- application splash screen with warnings
	- threat of demotion
	- front reception desk
	- posted warning signs

#### Detective Control Type
- detective
	- identify and log an intrusion attempt
	- may not prevent access
- find the issue
	- collect and review system logs
	- review login reports
	- regularly patrol the property
	- enable motion detectors

#### Corrective Control Type
- apply a control after an event has been detected
- reverse the impact of an event
- continue operating with minimal downtime

- correct a problem
	- restoring from backups can mitigate a ransomware infection
	- create policies for reporting security issues
	- contact law enforcement to manage criminal activity
	- use a fire extinguisher
#### Compensating control type
- control using other means
- existing controls aren't sufficient
- may be temporary

- prevent the exploitation of a weakness
	- firewall blocks a specific application instead of patching the app
	- implementing a separation of duties
	- require simultaneous guard duties
	- generator used after power outage

#### Directive control types
- direct a subject toward security compliance
- a relatively weak security control

- Do this, please
	- store all sensitive files in a protected folder
	- create compliance policies and procedures
	- train users on proper security policy
	- post a sign for "Authorized Personnel Only"

### Managing Security Tools
- these are not inclusive lists
	- there are many categories of control
	- some orgs will combine types

- there are multiple security controls for each category and type
	- some security controls may exist in multiple types or categories
	- new security controls are created as systems and processes evolve
	- your organization may use very different controls

# 1.2

## The CIA Triad
- Combination of principles
	- fundamentals of security
	- sometimes referenced as the AIC Triad

### Confidentiality
- prevent disclosure of information to unauthorized individuals
- certain information should only be known to certain people
	- prevents unauthorized information disclosure
- encryption
	- encode messages so only certain people can read it
- Access controls
	- selectively restrict access to a resource
- Two factor auth
	- additional confirmation before information is disclosed 
### Integrity
- messages can't be modified without detection
- data is stored and transferred as intended
	- any modification to the data would be identified
- hashing
	- map data of an arbitrary length to a data of fixed length
- digital signatures
	- mathematical scheme to verify integrity of data
	- hash and encrypts with an asymmetric encryption algorithm
- certificates
	- combine with a digital signature to verify an individual
- non-repudiation
	- provides proof integrity, can be asserted to be genuine
### Availability
- systems and networks must be up and running
- information is accessible to authorized users
	- always at your fingertips
- redundancy
	- build services that will always be available
- fault tolerance
	- system will continue to run, even when a failure occurs
- patching
	- stability
	- close security holes

## Non-repudiation
- you cant deny what you've said
	- there's no taking it back
- sign a contract
	- your signature adds non-repudiation
	- you really did sign the contract
	- others can see your signature
- Adds a different perspective for cryptography
	- proof of integrity
	- proof of origin, with high assurance of authenticity

### Proof of integrity
- verify data does not change
	- the data remains accurate and consistent
- in cryptography, we use a hash
	- represents data as a short string of text
	- a message digest, a fingerprint
- if the data changes, the hash changes
	- if the person changes, you get a different fingerprint
- doesn't necessarily associate data with an individual
	- only tells you if the data has changed

### Hashing the encyclopedia
- Lets say you were to download the text version of an encyclopedia (8.1 megabytes) and use an application to hash it.
	- djkaslfhuiopgnuiojp37yu8940432q89tpweriotnqwerntjhkrel;hqiopthruioepq89ut4prui
- Then change one character and rehash it again
	- fbnhdjluirnmkldsjkklfhjdskaljklgfjkdslnjgkl48u39p04hj4k3klh3jl4h3jkl43ghsduid9osaffhj
- If the hashes are different, something has changed
	- the data is compromised somewhere, run a diff to find it

### Proof of Origin
- prove  the message was not change
	- integrity
- prove the source of the message
	- authentication
- make sure the signature isn't fake
	- non-repudiation
- sign with the private key
	- the message doesn't need to be encrypted
	- nobody else can sign the (obviously)
- verify with the public key
	- any changes to the message will invalidate the signature

![[Screenshot 2026-05-30 220017 1.png]]

![[Screenshot 2026-05-30 220407.png]]

## Authentication, Authorization, and Accounting
Also Known as the AAA framework (triple a)
- identification
	- this is **who you say you are**
	- username
- authentication
	- **prove you are who you say you are**
	- password and other authentication factors
- authorization
	- based on your identification and authentication, **what access do you have?**
- accounting
	- resources used: login time, data sent and received, logout time

![[Screenshot 2026-05-31 094854.png]]

### Authentication systems
- you may have to manage many devices
	- often devices that you'll never physically see
- a system can't type a password
	- and you may not want to store one
- how can you truly authenticate a device?
	- generally, you put a digitally signed certificate on the device
- other business process rely on the certificate
	- access to the vpn from authorized devices
	- management software can validate the end device

### Certificate authentication
 - an org has a trusted certificate authority (CA)
	 - most orgs maintain their own CAs
- the organization creates a certificate for a device
	- and digitally signs the certificate with the orgs CA
- the certificate can now be included on a device as an authentication factor
	- the CAs digital signature is used to validate the certificate

### Authorization Model
- the user or device has now authenticated
	- to what do they have access?
	- time to apply an authorization model
- users and services -> data and applications
	- associating individual users to access rights does not scale
- put an authorization model in the middle
	- define by roles, organizations, attributes etc.

### No Authorization Model
- a simple relationship
	- user -> resource
- some issues with this method
	- difficult to understand why an authorization may exist
	- does not scale

![[Screenshot 2026-05-31 100543.png]]

### Using an authorization model
- add an abstraction
	- reduce complexity
	- create a clear relationship between the user and the resource
- administrator is streamlined
	- easy to understand the authorizations
	- support any number of users or resources

## Gap Analysis

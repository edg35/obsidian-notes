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

## Control Categories
**4 main control categories**
### Technical Controls
- Controls implemented using systems
- operating system controls
- firewalls, antivirus

### Managerial Controls
- administrative controls associated with security design and implementation
- security polices, standard operating procedures

### Operational Controls
- controls implemented by people instead of systems
- security guards, awareness programs

### Physical Controls
- Limit access
- guard shack
- fences, locks
- badge readers

## Control Types and Categories
**Control Type examples**

| Categories  | Preventative       | Deterrent      | Detective        | Corrective                   | Compensating                    | Directive                |
| ----------- | ------------------ | -------------- | ---------------- | ---------------------------- | ------------------------------- | ------------------------ |
| Technical   | Firewall           | Splash screen  | System logs      | Bakcup recovery              | Block instead of path           | file storage polices     |
| Managerial  | On-Boarding Policy | Demotion       | Login reports    | Policies for reporting issue | Separation of duties            | compliance policies      |
| Operational | Gaurd Shack        | Reception desk | Property Patrols | Contact authorities          | Require multiple security staff | security policy training |
| Physical    | Door Lock          | Warning sgns   | Motion detectors | Fire extinguisher            | Power generator                 | signs                    |

### Preventive Control Types
- preventive
	- block access to a resource
	- you shall not pass
- prevent access
	- firewall rules
	- follow security policy
	- guard shack checks all identification
	- enable door locks

### Deterrent Control Types
- deterrent
	- discourage an intrusion attempt
	- does not directly prevent access
- make and attacker think twice
	- application splash screen with warnings
	- threat of demotion
	- front reception desk
	- posted warning signs

### Detective Control Type
- detective
	- identify and log an intrusion attempt
	- may not prevent access
- find the issue
	- collect and review system logs
	- review login reports
	- regularly patrol the property
	- enable motion detectors

### Corrective Control Type
- apply a control after an event has been detected
- reverse the impact of an event
- continue operating with minimal downtime

- correct a problem
	- restoring from backups can mitigate a ransomware infection
	- create policies for reporting security issues
	- contact law enforcement to manage criminal activity
	- use a fire extinguisher
### Compensating control type
- control using other means
- existing controls aren't sufficient
- may be temporary

- prevent the exploitation of a weakness
	- firewall blocks a specific application instead of patching the app
	- implementing a separation of duties
	- require simultaneous guard duties
	- generator used after power outage

### Directive control types
- direct a subject toward security compliance
- a relatively weak security control

- Do this, please
	- store all sensitive files in a protected folder
	- create compliance policies and procedures
	- train users on proper security policy
	- post a sign for "Authorized Personnel Only"

## Managing Security Tools

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

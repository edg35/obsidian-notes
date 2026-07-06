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
- where you are comparted with where you want to be
	- the gap between the two
- This may require extensive research
- this can take weeks or months
	- extensive study with numerous participants
	- get ready for emails, data gathering and technical research

### Choosing the Framework
- work towards a known baseline
	-  this may be an internal set of goals
	- some orgs should use formal standards
- determine the end goal
	- NIST special publications
	- ISO/IEC

### Evaluate people and process
- get a baseline of employees
	- formal experience
	- current training
	- knowledge of polices
- examine the current processes
	- research existing IT systems and polices

### Compare and Contrast
- the comparison
	- evaluate existing systems
- identify weaknesses
- a detailed analysis

### The analysis and report
- the final comparison
	- detailed baseline vs where we are today
- need a path to get from the current security to the goal
- time to create the gap analysis report
	- formal description of current state

## Zero Trust
- many networks are relatively open on the outside
	- once you are through the firewall there are few security controls
- zero trust is a holistic approach to network security
- everything must b verified
	- nothing is trusted
	- multifactor, encryption, system permissions, additional; firewalls, monitoring and analytics

### Planes of operation
- split the network into functional planes
- data plane
	- process the frameworks, packets, and network data
	- processing, forwarding, trucking, encryption, NAT
- control plane
	- manage actions in data plane
	- policies and rules
	- how packets should be forwarded
	- routing tables, session, NAT tables

![[Screenshot 2026-06-19 090516.png]]

### Adaptive Identity

**Adaptive identity**: An authentication approach that evaluates contextual factors beyond just credentials to determine appropriate security controls.

Factors examined:

- Source IP address and geolocation (e.g., US-based resource accessed from a Chinese IP)
- Relationship to the organization (employee, contractor, full-time, part-time)
- Physical location
- Type of connection
- Any other available contextual signals
- The system automatically triggers stronger authentication when contextual risk is elevated

### Limiting Entry Points / Threat Scope Reduction

- Restrict network entry to specific methods only (e.g., on-premises access or VPN — no other methods)
- Reducing entry points limits the attack surface available to unauthorized users

### Policy-Driven Access Control

**Policy-driven access control**: Aggregates all identity and context data points to determine the appropriate authentication method for a given access request.

### Security Zones

**Security zones**: Network segments categorized by trust level, used to define access rules based on where a connection originates and where it is trying to reach.

Common zone types:

- Untrusted (external/internet
- Trusted (corporate offices, internal users)
- Internal (data centers, servers)
- VPN or department-specific zones
- Rules can explicitly deny traffic from untrusted → trusted zones
- **Implicit trust**: Communication between two trusted zones (e.g., trusted → internal) can be automatically allowed by policy

### Zero Trust Policy Components

**Policy Enforcement Point (PEP)**: Acts as the network gatekeeper — all traffic passes through it. Collects information about traffic/users but does not make allow/deny decisions itself. Forwards that data to the Policy Decision Point.

**Policy Decision Point (PDP)**: Responsible for evaluating authentication and deciding whether traffic should be allowed. Composed of two sub-components:

- **Policy Engine**: Compares incoming requests against predefined security policies and outputs a decision: grant, deny, or revoke.
- **Policy Administrator**: Receives the Policy Engine's decision and communicates it back to the PEP. Creates and distributes any resulting access tokens or credentials.

### Zero Trust Data Flow (End-to-End)

1. Subject/system in an **untrusted zone** initiates a request over the **data plane**
2. Traffic passes through the **Policy Enforcement Point (PEP)**
3. PEP forwards context to the **Policy Administrator**
4. Policy Administrator relays it to the **Policy Engine**
5. Policy Engine evaluates against security policies → issues grant/deny/revoke
6. Decision flows back: Policy Engine → Policy Administrator → PEP
7. If granted, PEP allows access to the **trusted zone** and the requested **Enterprise Resource**

#### Acronyms

|Acronym|Full Name|Description|
|---|---|---|
|PEP|Policy Enforcement Point|Gatekeeper that collects traffic info and enforces access decisions|
|PDP|Policy Decision Point|Evaluates requests; contains the Policy Engine and Policy Administrator|
|NAT|Network Address Translation|Remaps IP addresses; configured in the control plane|
|MFA|Multi-Factor Authentication|Required login step in zero trust environments|
|VPN|Virtual Private Network|Controlled entry point in a zero trust network|

## Physical Security Controls

### Barricades and Bollards

**Bollard/barricade**: A physical barrier used to channel people through designated access points and prevent vehicles from entering a secured area.

- Often made of concrete; can also be water barriers requiring bridge access
- Brightly colored bollards serve as a visible security notice for high-security areas
- Allow pedestrian passage while blocking cars, trucks, and other vehicles

### Access Control Vestibule

**Access control vestibule**: A room that must be passed through to gain access to the rest of a building; controls entry by ensuring only one door can be open at a time.

Three common configurations:

1. **All doors normally unlocked** — opening one door prevents others from opening while it remains open
2. **All doors normally locked** — badging in unlocks the first door; all others remain locked until it closes
3. **One door always locked, one always unlocked** — opening the unlocked door prevents the locked door from being opened

- May allow one person at a time or a controlled group
- Common in large data centers and high-security facilities
- Often combined with card/biometric readers and a security checkpoint inside

### Fencing

- Provides a visible, robust perimeter barrier
- **Transparent fence**: allows visibility of what's beyond it
- **Opaque fence**: conceals what's on the other side
- High-security fencing may include increased height and razor wire to prevent climbing

### Cameras / CCTV

**CCTV (Closed Circuit Television)**: A networked system of cameras that feed video to a single centralized storage point for recording and monitoring.

- Modern cameras include **motion detection**, **object detection**, **facial recognition**, and **license plate reading**
- All cameras send video to one storage location for continuous recording

#### Acronyms

|Acronym|Full Name|Description|
|---|---|---|
|CCTV|Closed Circuit Television|Private camera network; feeds to centralized storage|

### Security Guards

- Validate that individuals entering a building are employees or authorized guests
- Often work in pairs for **two-person integrity** — ensures no single guard can unilaterally circumvent security policy; provides checks and balances

### Identification Badges

- Worn visibly at all times (lanyard or attached to clothing)
- Contains photo, name, and individual details
- Integrated with electronic door locks — badge access events are logged to a central database

### Lighting

- Illuminating dark areas deters unauthorized entry — attackers prefer to operate out of sight
- Proper lighting angles matter for camera-based facial recognition
- Parking lots: lighting + 24/7 camera monitoring is a common combination

### Sensors and Motion Detection

**Infrared sensors**: Detect infrared radiation to identify motion in both lit and dark environments; used in cameras and motion detectors. Best suited for limited/small areas.

**Pressure sensors**: Detect changes in force as someone moves across a surface; triggers an alert when pressure is sensed.

**Microwave sensors**: Detect motion over large areas; more effective than infrared for wide-area coverage.

**Ultrasonic sensors**: Emit ultrasonic signals and detect reflected sound waves to identify motion; can also be used for collision detection in parking lots or loading zones.

#### Sensor Comparison

|Sensor Type|Detection Method|Best Use Case|
|---|---|---|
|Infrared|Infrared radiation|Small/limited areas, dark environments|
|Pressure|Force/weight change|Floor-based intrusion detection|
|Microwave|Microwave signal reflection|Large open areas|
|Ultrasonic|Sound wave reflection|Motion + collision detection|
## Deception and Disruption Technologies

### Honeypots

**Honeypot**: A decoy system designed to attract attackers and observe their techniques without exposing real production systems.

- Attackers (usually automated processes) interact with fake systems while defenders monitor what attack methods and automation are being used
- Creates a race between defenders making honeypots more realistic and attackers developing techniques to detect them
- Built using commercial or open-source software
- Resource: projecthoneypot.org

### Honeynets

**Honeynet**: A larger, more complex deception environment built by combining multiple honeypots into a full fake infrastructure.

- May include workstations, servers, routers, and firewalls to appear as a legitimate network
- More believable than a single honeypot; keeps attackers occupied longer

### Honeyfiles

**Honeyfile**: A fake file placed on a system containing fabricated or worthless data, designed to look attractive to an attacker.

- Example: a file named `passwords.txt` that contains no real credentials
- No legitimate users should ever access these files in normal operation
- Accessing or opening a honeyfile triggers an alert/alarm to a management station — indicates unauthorized activity

### Honeytokens

**Honeytoken**: A piece of traceable fake data placed in a system or network; if the data is copied and distributed externally, defenders can trace its origin and identify the attacker.

Examples of honeytokens:

- **Fake API credentials** posted to a public cloud share — not real/usable, but monitored for use
- **Fake email addresses** — monitored for appearance elsewhere on the internet; can identify who leaked them
- **Database records**, **browser cookies**, **web page pixels**, or any other falsifiable data that can be tracked if redistributed

### Key Distinctions

|Term|Scope|Purpose|
|---|---|---|
|Honeypot|Single decoy system|Attract and observe attackers|
|Honeynet|Full fake infrastructure (multiple honeypots)|More realistic environment to engage attackers|
|Honeyfile|Fake file on a system|Detect unauthorized file access|
|Honeytoken|Traceable fake data|Identify data exfiltration and trace attacker identity|

# 1.3

## Change Management

### Overview

**Change management**: A formal process for controlling changes to systems, applications, and infrastructure in an organization to maintain uptime, availability, and security.

- Uncontrolled changes can cause application inconsistencies, outages, and security gaps
- Defines how often changes can be made, what types are allowed, and rollback procedures
- Should be documented in standard operating procedures accessible to all staff
- No changes should be made without formal approval

### Change Control Process (Typical Steps)

1. **Submit a change control form** — standardized form submitted to a central change control board (CCB)
2. **Document the reason** for the change
3. **Define the scope** — single system or multiple systems
4. **Schedule the change** — specify date and time
5. **Identify affected systems** and the impact of the change
6. **Risk analysis** — CCB weighs risk of making vs. not making the change
7. **CCB decision** — change is approved or denied
8. **Implement the change**
9. **User testing and verification** — confirm no issues after the change

### Roles in Change Management

**Owner**: The person or department that owns the application or data being changed.

- Initiates the change request
- Does NOT control the change process or make the change themselves
- Kept informed throughout the process
- Responsible for testing and verifying the system after the change is complete

**Stakeholders**: Individuals or departments impacted by a proposed change.

- May not be immediately obvious — changes can have unexpected downstream effects
- Should have input on the process and scheduling
- IT must proactively identify all stakeholders

> Example: Upgrading shipping label software seems isolated but can affect accounting reports, delivery timelines, revenue recognition, and executive visibility.

### Risk Analysis

Every change carries risk in both directions:

**Risks of making the change:**
- Fix doesn't resolve the problem
- Update breaks something else
- OS failure caused by the patch
- Data corruption (requires backups)

**Risks of NOT making the change:**

- Security vulnerability remains exploitable
- Application becomes unavailable
- Dependent services stop functioning
- Risk is typically categorized as **high, medium, or low**
### Sandbox Testing

**Sandbox**: An isolated test environment that mirrors production; used to safely test changes with no impact on live systems.

- Load a duplicate of the production environment
- Apply the patch/change and observe results
- Test **backout procedures** even if the change works — confirms rollback steps are valid before going to production

### Backout Plan

**Backout plan**: A documented set of steps to revert a system to its original state if a change causes problems.

- Always take a **full backup** before making any change — last resort if both the change and the backout plan fail
- Simple backouts: uninstall the patch and verify original files are restored
- Complex changes may require additional reversion techniques
- Many outages have occurred because someone skipped a backout plan for a "minor" change

### Change Scheduling

- Avoid changes during business hours when users are active
- Use **off-hours or maintenance windows** (nights, weekends, holidays)
- 24/7 environments have very limited maintenance windows
- **Change freezes**: Some organizations (e.g., retail) freeze all changes during peak seasons (e.g., Thanksgiving–New Year's)

### Acronyms

|Acronym|Full Name|Description|
|---|---|---|
|CCB|Change Control Board|Committee responsible for evaluating and approving/denying change requests|

## Technical Change Management

### Allow Lists and Deny Lists

**Allow list**: Only applications explicitly named on the list are permitted to run — everything else is blocked by default.

**Deny list**: All applications are permitted to run except those explicitly named — more flexible than an allow list.

- Antivirus/anti-malware is conceptually a large deny list — everything runs except code identified as malicious
- Technicians may be tasked with adding/removing entries from either list as part of a change control

### Scope of Change

- Technicians are limited to changes explicitly listed in the approved change control document
- Do **not** use a maintenance window to make unrelated changes
- **Scope creep exception**: If completing the primary change requires an undocumented secondary change (e.g., modifying a config file to apply a driver update), policies may allow the technician to make that call — must be well-documented

### Downtime and Maintenance Windows

- Changes don't always cause downtime, but a window is typically reserved to communicate potential unavailability
- Changes should be made during **non-production/off hours** whenever possible
- **24/7 environments**: Use a primary/secondary failover approach:
    1. Switch users to the secondary system
    2. Update the primary system
    3. Switch back seamlessly (usually automated)
    4. If problems arise, switch back to the unchanged secondary system
- Notify stakeholders of potential downtime via email or a centralized change control calendar

### Reboots and Service Restarts

Three levels of restart depending on the change:

1. **Full system reboot** — required when a new config only takes effect on restart; also validates power-cycle recovery behavior
2. **Service/daemon restart** — faster; done via Windows Services, Task Manager, or Linux daemon commands
3. **Application restart** — users close and reopen the app to load the updated executable

### Legacy Applications

**Legacy application**: An application that has been running for a long time, has no planned replacement, and is often no longer supported by the original developer.

- Often undocumented with informal warnings like "don't touch this system"
- Approach: Document how the application is installed and configured to bring it into the normal support cycle
- May have idiosyncrasies specific to running on older OSes
- Documenting the system makes the technician the internal expert and improves org-wide supportability

### Dependencies

**Dependency**: A condition where one application or service must be updated or installed before another can be installed or updated.

- Complicates change control — a single change may require updating multiple services
- Dependencies can exist **across systems** (e.g., firewall management software requires all firewalls to be on a new code version first)
- Must be identified and sequenced before the change window begins

### Documentation Requirements

- Every change potentially makes existing documentation outdated
- Change management processes should **require documentation updates** as part of the change itself
- Documentation to update may include:
    - Network diagrams and topology charts
    - IP address records and configurations
    - Procedures for managing new/upgraded applications

### Version Control

**Version control**: A system for tracking changes to code, configurations, and software over time; enables rollback to a previous version if a change causes issues.

- Tracks changes to router configs, OS patches, registry changes, application updates, etc.
- Some applications and OSes have version control built in
- Third-party version control systems can be used when native support is absent

#### Key Terms Summary

|Term|Definition|
|---|---|
|Allow list|Only listed apps are permitted to run|
|Deny list|All apps permitted except those listed|
|Legacy application|Long-running, often unsupported app with no planned replacement|
|Dependency|Prerequisite service/app that must be updated before the target change|
|Version control|System for tracking and reverting changes to configs/software/code|

# 1.4

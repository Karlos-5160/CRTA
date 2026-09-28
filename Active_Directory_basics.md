**Active Directory (AD)** is a combination of:

**a database + authentication system + authorization system + DNS-based network directory + centralized management system.**

Think of a company with 5,000 Windows computers. You do **not** want every computer to maintain separate users, passwords, policies, permissions, and network information. Active Directory centralizes that.

---

# 1. What is Active Directory?

**Active Directory** is Microsoft's directory service used primarily in Windows enterprise networks.

The most important implementation is:

> **Active Directory Domain Services (AD DS)**

AD DS stores information about objects such as:

* Users
* Computers
* Groups
* Printers
* Servers
* Organizational Units
* Service accounts
* Other network resources

It also provides:

* Authentication — **Who are you?**
* Authorization — **What are you allowed to access?**
* Centralized management — **What rules should apply to your computer/user?**
* Resource discovery — **Where is the service/computer I need?**

### Simple analogy

Imagine a university.

There are:

* 10,000 students
* 500 teachers
* 300 computers
* labs
* departments
* servers
* printers

Instead of every computer knowing every person's username and password, the university maintains a **central identity system**.

That identity system is similar to Active Directory.

---

# 2. Active Directory is NOT just "a database"

This is a very important point.

People often say:

> "Active Directory is a database."

That's only partially correct.

AD contains a directory database, but AD DS also provides the services needed to use that information.

For example:

```text
User
  |
  | "I want to log in"
  v
Active Directory
  |
  +--> Is this user valid?
  |
  +--> Is the password correct?
  |
  +--> Which groups is the user in?
  |
  +--> What permissions should the user receive?
```

So AD is better thought of as:

```text
Active Directory
│
├── Directory database
├── Authentication
├── Authorization information
├── DNS integration
├── Group Policy
├── Replication
├── Trust relationships
└── Centralized administration
```

---

# 3. The most important terms

Before going deeper, memorize these:

| Term                   | Meaning                                                       |
| ---------------------- | ------------------------------------------------------------- |
| AD                     | Active Directory                                              |
| AD DS                  | Active Directory Domain Services                              |
| Domain                 | Administrative/security boundary containing AD objects        |
| Domain Controller (DC) | Server running AD DS                                          |
| Forest                 | Largest AD logical/security boundary                          |
| Tree                   | Collection of related domains                                 |
| OU                     | Organizational Unit used to organize/manage objects           |
| Schema                 | Defines what object types and attributes can exist            |
| Global Catalog         | Helps search across the forest                                |
| GPO                    | Group Policy Object                                           |
| DNS                    | Helps clients find domain services/DCs                        |
| Kerberos               | Main authentication protocol                                  |
| NTLM                   | Older authentication protocol still supported                 |
| SID                    | Security Identifier                                           |
| RID                    | Relative Identifier                                           |
| SPN                    | Service Principal Name                                        |
| LDAP                   | Protocol used to access directory information                 |
| SYSVOL                 | Shared folder containing important domain policy/script files |
| FSMO                   | Five special AD operation-master roles                        |

---

# 4. The Active Directory hierarchy

One of the biggest sources of confusion is this:

```text
Forest
   |
   +-- Tree
        |
        +-- Domain
             |
             +-- OU
                  |
                  +-- Users
                  +-- Computers
                  +-- Groups
```

Let's understand each.

---

# 5. Forest

A **forest** is the highest-level logical structure in Active Directory.

Example:

```text
Forest
└── company.com
```

Suppose a company has:

```text
company.com
```

That could be one forest containing one domain.

But a large organization could have:

```text
Forest
│
├── company.com
│
├── europe.company.com
│
└── asia.company.com
```

More precisely, domains arranged under a common namespace form trees, and multiple trees can exist inside one forest.

### Key idea

The **forest is the ultimate AD security boundary** in the traditional AD DS architecture.

The forest contains shared information such as:

* Schema
* Configuration
* Global Catalog
* Trust relationships between domains

---

# 6. Tree

A **tree** is a group of domains that share a contiguous DNS namespace.

Example:

```text
company.com
    |
    +-- india.company.com
    |
    +-- us.company.com
    |
    +-- europe.company.com
```

These domains form a tree because they share:

```text
company.com
```

Another tree could exist in the same forest:

```text
company.com
   |
   +-- ...

research.org
   |
   +-- ...
```

Both could belong to the same forest even though their DNS namespaces are different.

---

# 7. Domain

A **domain** is one of the most important concepts in AD.

Example:

```text
corp.example.com
```

A domain contains objects like:

```text
Users
Computers
Groups
Servers
OUs
```

For example:

```text
corp.example.com
│
├── Alice
├── Bob
├── DC01
├── PC01
├── HR
├── IT
└── Domain Admins
```

### What does a domain actually do?

It provides a common security and administrative boundary.

For example, if:

```text
alice@corp.example.com
```

logs into:

```text
PC01
```

PC01 can ask the domain controller whether Alice is a valid domain user.

---

# 8. Domain Controller (DC)

This is extremely important.

A **Domain Controller** is a Windows Server running AD DS.

It contains a copy of the directory database.

The main AD database file is:

```text
C:\Windows\NTDS\ntds.dit
```

Think of it as:

```text
Domain
    |
    +------ DC01
    |
    +------ DC02
    |
    +------ DC03
```

Each DC maintains directory information and participates in replication.

Therefore, if:

```text
DC01
```

creates:

```text
user = kuldeep
```

that information can replicate to:

```text
DC02
DC03
```

---

# 9. Why have multiple Domain Controllers?

Imagine there is only one DC:

```text
Users
   |
   v
 DC01
```

What happens if DC01 dies?

Authentication and many domain services can be severely disrupted.

Therefore:

```text
             +-- DC01
             |
Users -------+
             |
             +-- DC02
             |
             +-- DC03
```

Multiple DCs provide:

* Redundancy
* Availability
* Load distribution
* Replication

This is why AD is generally designed with multiple DCs.

---

# 10. RODC — Read Only Domain Controller

There is also:

> **RODC = Read-Only Domain Controller**

It contains a read-only copy of directory information.

Useful in locations where physical security is weaker, such as:

```text
Head Office
     |
     +------ DC01

Remote Branch
     |
     +------ RODC
```

An attacker who compromises the RODC has a different situation compared with compromising a writable DC.

---

# 11. Organizational Unit (OU)

This causes lots of confusion.

An **OU is mainly an organizational and administrative container**.

For example:

```text
company.com
│
├── OU=HR
│     ├── Alice
│     └── Bob
│
├── OU=IT
│     ├── John
│     └── Sarah
│
└── OU=Finance
      ├── David
      └── Mike
```

OUs are useful for:

* Organizing objects
* Delegating administration
* Applying Group Policy

### Very important:

> **An OU is NOT automatically a security boundary.**

You don't normally put users in an OU simply because you want to isolate them like a firewall network.

---

# 12. Containers vs OUs

AD has other containers too.

For example:

```text
Users
Computers
Builtin
```

These are containers.

An OU is more useful for applying administrative policies such as GPOs.

---

# 13. Objects in Active Directory

Everything in AD is represented as an object with attributes.

For example:

```text
User object
```

might have:

```text
username = alice
displayName = Alice Smith
email = alice@example.com
memberOf = HR
employeeID = 1234
```

A computer object could have:

```text
Computer = WIN-PC01
DNS name = win-pc01.company.com
Operating System = Windows
```

---

# 14. Attributes

Objects contain **attributes**.

Think:

```text
Object
  |
  +-- attribute
  +-- attribute
  +-- attribute
```

Example:

```text
User: Alice

sAMAccountName = alice
userPrincipalName = alice@company.com
displayName = Alice Smith
mail = alice@company.com
memberOf = HR
objectSid = S-1-5-21-...
```

This leads us to an important concept:

# 15. AD Schema

The **Schema** defines what types of objects can exist and which attributes they can have.

Think of the schema as the **blueprint** of Active Directory.

Example:

```text
Schema says:

User objects can have:
    username
    email
    display name
    SID
    password-related data
    etc.

Computer objects can have:
    computer name
    OS information
    SID
    etc.
```

The schema is forest-wide.

---

# 16. LDAP

Another major concept:

> **LDAP = Lightweight Directory Access Protocol**

LDAP is a protocol used to interact with directory information.

You can think of it like:

```text
Application
     |
     | LDAP request
     v
Active Directory
     |
     +--> Search users
     +--> Find groups
     +--> Read attributes
     +--> Modify objects
```

Typical LDAP port:

```text
389
```

LDAP over TLS:

```text
636
```

In AD environments, LDAP is deeply integrated with directory operations.

---

# 17. LDAP vs Active Directory

These are NOT the same thing.

Think:

```text
LDAP = protocol

Active Directory = Microsoft's directory service
```

Just like:

```text
HTTP = protocol
Apache = web server
```

AD can use LDAP to provide directory access.

---

# 18. DNS and Active Directory

This is one of the most important relationships in AD.

People often think:

> "DNS is only for converting names to IP addresses."

In AD, DNS does more than that.

Clients use DNS to locate domain services.

For example:

```text
Where is a domain controller for example.com?
```

DNS can provide service records such as:

```text
_ldap._tcp
_kerberos._tcp
```

So:

```text
Client
  |
  | DNS query
  v
DNS Server
  |
  +--> Find appropriate DC
  |
  v
Domain Controller
```

This is why a broken DNS configuration can make an AD environment appear completely broken.

---

# 19. Domain Name vs NetBIOS Domain Name

Suppose the AD domain is:

```text
corp.example.com
```

Its old-style NetBIOS name might be:

```text
CORP
```

So you may see:

```text
CORP\alice
```

or:

```text
alice@corp.example.com
```

These can represent the same user.

---

# 20. UPN

UPN means:

> **User Principal Name**

Format:

```text
username@domain
```

Example:

```text
alice@corp.example.com
```

This is a common way of identifying a domain user.

---

# 21. Domain users vs local users

This distinction is extremely important in Windows security.

### Local user

Exists only on one machine:

```text
PC01
 └── localuser
```

Its account is stored locally.

### Domain user

Exists in AD:

```text
Domain
 └── alice
```

That user can potentially log into many domain-joined machines depending on permissions/policies.

Therefore:

```text
LOCAL\alice
```

and

```text
CORP\alice
```

can be completely different accounts.

---

# 22. Joining a computer to a domain

Suppose you have:

```text
PC01
```

and domain:

```text
corp.example.com
```

When PC01 joins the domain, AD creates a **computer object**.

Conceptually:

```text
AD
 |
 +-- PC01$
```

Notice the `$`.

Computer accounts traditionally use names ending in:

```text
$
```

Example:

```text
PC01$
DC01$
WEB01$
```

---

# 23. Computer accounts are accounts too

This is a major AD concept.

Many beginners think:

> "Only people have AD accounts."

No.

Machines have accounts too.

For example:

```text
PC01$
WEB01$
DC01$
```

A computer account has credentials and participates in authentication.

This becomes especially important in:

* Kerberos
* Machine authentication
* Domain trust
* Secure channels
* Delegation

---

# 24. Secure Channel

A domain-joined machine maintains a secure relationship with a DC.

Conceptually:

```text
PC01  <====== secure relationship ======>  DC01
```

This is often called the **machine secure channel**.

Tools and Windows mechanisms can test/repair this relationship.

---

# 25. Authentication vs Authorization

This distinction is fundamental.

### Authentication

> **Who are you?**

Example:

```text
Username: Alice
Password: ********
```

System verifies Alice.

### Authorization

> **What are you allowed to do?**

Example:

```text
Alice
  |
  +--> Read HR share
  +--> Cannot access Finance share
  +--> Can print
  +--> Cannot install software
```

Remember:

```text
Authentication = identity
Authorization = permissions
```

---

# 26. Kerberos

In modern AD environments, **Kerberos is the primary authentication protocol**.

You will encounter it constantly in Active Directory labs.

The simplified flow is:

```text
User
 |
 | 1. Authentication request
 v
KDC
 |
 | 2. TGT
 v
User
 |
 | 3. Service ticket request
 v
KDC
 |
 | 4. Service ticket
 v
Server
```

The KDC is normally provided by the domain controller.

KDC means:

> **Key Distribution Center**

It has two conceptual services:

```text
AS = Authentication Service
TGS = Ticket Granting Service
```

---

# 27. Kerberos step-by-step

Suppose:

```text
alice
```

wants to access:

```text
\\FILE01\HR
```

### Step 1 — Alice authenticates

Alice's machine contacts the KDC.

Conceptually:

```text
Alice
  |
  | "I am Alice"
  v
KDC
```

The KDC verifies the request.

---

### Step 2 — TGT

The KDC gives Alice a:

> **TGT = Ticket Granting Ticket**

Think of the TGT as:

> "Alice has successfully authenticated."

It is not the ticket to every service.

It's more like a credential that allows Alice to request service tickets.

---

# 28. TGS / service ticket

Alice wants:

```text
FILE01
```

Her machine asks the KDC for a service ticket.

Conceptually:

```text
Alice
  |
  | "Give me a ticket for FILE01"
  v
KDC
  |
  v
Service Ticket
```

Then:

```text
Alice ---> FILE01
       service ticket
```

The server can validate that ticket.

---

# 29. SPN

This is another critical AD term:

> **SPN = Service Principal Name**

An SPN identifies a service instance for Kerberos.

For example, conceptually:

```text
HTTP/webserver.company.com
MSSQLSvc/sql.company.com:1433
```

Kerberos needs to know:

> "Which account owns this service?"

SPNs connect services with accounts.

This becomes extremely important in:

* Kerberos authentication
* Service accounts
* Delegation
* Kerberoasting

---

# 30. NTLM

Another Windows authentication system is:

> **NTLM**

Historically important and still encountered in AD environments.

Simplified idea:

```text
Client ---> Server
           "Give me a challenge"

Client ---> Server
           "Here's my response"

Server ---> DC
           "Is this valid?"
```

NTLM relies on challenge-response rather than the Kerberos ticket model.

In modern AD environments:

```text
Kerberos = preferred/primary
NTLM = legacy/fallback in many situations
```

---

# 31. LSASS

On Windows, a very important security process is:

> **LSASS = Local Security Authority Subsystem Service**

Process:

```text
lsass.exe
```

It handles important security functions including authentication-related operations.

From a cybersecurity perspective, LSASS is extremely important because authentication secrets and security material can be associated with its memory and Windows security mechanisms.

This is why defenders pay close attention to access to:

```text
lsass.exe
```

---

# 32. SID

Every security principal has a:

> **SID = Security Identifier**

Example:

```text
S-1-5-21-111111111-222222222-333333333-1105
```

Windows uses SIDs rather than simply relying on names for security decisions.

For example:

```text
Alice
   |
   v
SID
   |
   v
Permissions
```

---

# 33. Why does Windows use SIDs?

Suppose a user is named:

```text
Alice
```

Later the account is deleted and another account called Alice is created.

Windows should not automatically consider them the same security principal.

Their SIDs would be different.

Therefore:

```text
Account name = human-readable identity
SID          = security identity
```

---

# 34. Domain SID and RID

A typical domain user SID looks conceptually like:

```text
S-1-5-21-A-B-C-1105
```

The first part:

```text
S-1-5-21-A-B-C
```

is associated with the domain.

The final number:

```text
1105
```

is the:

> **RID = Relative Identifier**

So:

```text
Domain SID + RID = User SID
```

---

# 35. RID Master

In a domain, a special FSMO role called the:

> **RID Master**

helps manage allocation of RID pools to domain controllers.

Imagine:

```text
RID Master
   |
   +--> DC01 gets RID range
   +--> DC02 gets RID range
   +--> DC03 gets RID range
```

This allows DCs to efficiently create unique security principals.

---

# 36. Security groups

Groups are one of the most important things in AD.

Instead of giving permissions individually:

```text
Alice -> Read
Bob   -> Read
John  -> Read
Sarah -> Read
```

you create:

```text
HR-Read
```

Then:

```text
Alice
Bob
John
Sarah
   |
   v
HR-Read
   |
   v
HR share
```

Much easier to manage.

---

# 37. Types of groups

Two important group purposes are:

### Security groups

Used for permissions.

Example:

```text
Domain Admins
HR-Users
IT-Admins
```

### Distribution groups

Primarily used for email/distribution purposes rather than access control.

---

# 38. Group scopes

AD commonly uses three security-group scopes:

### Domain Local

Primarily used to assign permissions to resources in its domain.

### Global

Normally contains users/groups from its own domain and can be used across trusted domains.

### Universal

Can contain members across domains in the forest and is useful for forest-wide group organization.

---

# 39. AGDLP

You'll see this concept frequently in AD administration.

> **AGDLP**

Means:

```text
A = Accounts
G = Global groups
DL = Domain Local groups
P = Permissions
```

Conceptually:

```text
Users
  |
  v
Global Group
  |
  v
Domain Local Group
  |
  v
Permission
```

Example:

```text
Alice
Bob
   |
   v
GG_HR
   |
   v
DL_HR_File_Read
   |
   v
HR Share
```

This makes permissions easier to manage.

---

# 40. Domain Admins

One of the most sensitive groups in a traditional AD domain is:

```text
Domain Admins
```

A member of Domain Admins typically has extensive administrative control over the domain.

But don't confuse:

```text
Domain Admin
```

with:

```text
Local Administrator
```

They are different concepts.

---

# 41. Enterprise Admins

Another extremely sensitive group is:

```text
Enterprise Admins
```

This operates at the forest level and is associated with highly privileged administrative operations across domains in the forest.

---

# 42. Global Catalog

The:

> **Global Catalog (GC)**

is a searchable, forest-wide directory service.

Imagine the forest contains:

```text
india.company.com
uk.company.com
usa.company.com
```

A GC helps users and applications locate objects throughout the forest.

A GC stores a **partial set of attributes** for objects from the forest rather than simply being a full writable copy of every domain database.

Common GC ports include:

```text
3268 = LDAP Global Catalog
3269 = Global Catalog over TLS
```

---

# 43. Group Policy

Now we reach one of AD's most powerful features:

> **Group Policy**

Group Policy lets administrators centrally control Windows settings.

For example:

```text
Users
   |
   v
Domain
   |
   v
Group Policy
   |
   +--> Password policy
   +--> Firewall settings
   +--> Software restrictions
   +--> Desktop settings
   +--> Scripts
   +--> Security settings
```

---

# 44. GPO

A:

> **GPO = Group Policy Object**

contains configuration settings.

Examples:

```text
Disable USB storage
Configure Windows Defender
Set password requirements
Configure firewall
Map network drive
Run logon scripts
```

GPOs can target:

* Sites
* Domains
* OUs

---

# 45. GPO processing order

A useful rule to remember is:

```text
L → S → D → OU
```

Meaning:

```text
Local
Site
Domain
Organizational Unit
```

Then nested OUs are processed along the OU path.

People often call this:

> **LSDOU**

When conflicting settings exist, later applicable policy generally has precedence, subject to exceptions such as enforced/blocked inheritance and other policy mechanics.

---

# 46. GPC and GPT

A GPO is represented using two important pieces:

### GPC

> Group Policy Container

Stored in Active Directory.

### GPT

> Group Policy Template

Stored in:

```text
SYSVOL
```

Conceptually:

```text
GPO
├── GPC -> AD
└── GPT -> SYSVOL
```

This distinction is useful when troubleshooting Group Policy.

---

# 47. SYSVOL

Every normal writable DC maintains a:

```text
SYSVOL
```

shared area.

It contains domain-wide files such as:

* Group Policy files
* Logon/startup scripts
* Other replicated domain content

Conceptually:

```text
DC01
 └── SYSVOL
      ├── Policies
      └── Scripts
```

SYSVOL is replicated between DCs.

---

# 48. Replication

Suppose you create:

```text
Alice
```

on DC01.

How does DC02 learn about Alice?

Through:

> **AD replication**

Conceptually:

```text
             AD
              |
       +------+------+
       |             |
      DC01          DC02
       |             |
       +------sync---+
```

Modern AD uses **multi-master replication**, meaning writable DCs can generally accept changes rather than having one universal "master DC" for all directory writes.

---

# 49. Why replication matters

Without replication:

```text
DC01 knows Alice
DC02 doesn't
DC03 doesn't
```

That would produce inconsistent authentication and directory information.

With replication:

```text
DC01 ---> DC02
  \------> DC03
```

the directory can converge across domain controllers.

---

# 50. AD partitions

AD isn't simply one giant database logically.

Important directory partitions include:

### Domain partition

Contains domain-specific objects.

```text
Users
Computers
Groups
```

### Configuration partition

Contains forest-wide configuration information.

### Schema partition

Contains the schema definitions.

There can also be application partitions.

---

# 51. FSMO roles

Although AD uses multi-master replication, some operations need a single authority.

These are called:

> **FSMO roles**

There are **five**.

Two forest-wide:

```text
1. Schema Master
2. Domain Naming Master
```

Three domain-wide:

```text
3. RID Master
4. PDC Emulator
5. Infrastructure Master
```

---

# 52. Schema Master

Responsible for changes to the AD schema.

For example:

```text
"Add a new object class"
"Add a new attribute"
```

That's a schema-level operation.

There is one Schema Master per forest.

---

# 53. Domain Naming Master

Controls adding/removing domains and related namespace operations within the forest.

One per forest.

---

# 54. RID Master

Responsible for RID allocation to DCs.

Remember:

```text
Domain SID + RID = object SID
```

---

# 55. PDC Emulator

Despite its name, it is not simply an old "primary domain controller."

It has important functions such as:

* Time synchronization within the domain hierarchy
* Important password-change handling
* Certain account lockout/authentication behaviors
* Compatibility functions

It is extremely important operationally.

### Time is especially important

Kerberos is sensitive to clock differences.

So:

```text
Client time
      |
      | must be close
      v
DC time
```

Incorrect time synchronization can break Kerberos authentication.

---

# 56. Infrastructure Master

Handles certain cross-domain object-reference updates.

It is usually less discussed when first learning AD, but it is one of the five FSMO roles.

---

# 57. AD Sites

Now let's move from the logical structure to physical/network structure.

An AD:

> **Site**

represents a network location, generally based on IP subnets and network connectivity.

Example:

```text
Site: India
   10.10.0.0/16

Site: USA
   10.20.0.0/16
```

Sites help AD understand network topology.

---

# 58. Why sites exist

Imagine:

```text
India DC
    |
    | 200 Mbps WAN
    |
USA DC
```

You don't necessarily want constant heavy replication over a slow WAN.

Sites help AD optimize:

* Replication
* Client/DC selection
* Network traffic

---

# 59. Authentication from a user's perspective

Let's put many concepts together.

Suppose:

```text
User = Alice
Domain = corp.example.com
PC = PC01
```

Alice logs into PC01.

### Step 1

PC01 needs to find a domain controller.

It uses:

```text
DNS
```

### Step 2

PC01 contacts a DC.

### Step 3

Kerberos authentication occurs.

### Step 4

Alice receives authentication credentials/tickets.

### Step 5

Windows builds an **access token**.

This is important.

---

# 60. Access Token

Windows uses an access token to represent the user's security context.

It can include:

```text
User SID
Group SIDs
Privileges
Other security information
```

Conceptually:

```text
Alice
 |
 v
Access Token
 |
 +-- Alice SID
 +-- Domain Users SID
 +-- HR SID
 +-- Other group SIDs
 +-- privileges
```

When Alice tries to access a resource, Windows uses that security context to make authorization decisions.

---

# 61. ACL, ACE, DACL, SACL

These four terms are very important.

### ACL

> Access Control List

A list describing security permissions.

### ACE

> Access Control Entry

One individual permission rule in an ACL.

### DACL

> Discretionary Access Control List

Controls who is allowed/denied access.

### SACL

> System Access Control List

Controls auditing rules.

Conceptually:

```text
File
 |
 +-- DACL
 |    |
 |    +-- Alice -> Read
 |    +-- HR -> Modify
 |    +-- Everyone -> Deny
 |
 +-- SACL
      |
      +-- Audit access
```

---

# 62. How authorization actually works

Suppose:

```text
Alice
```

tries to access:

```text
\\FILE01\HR
```

Windows can conceptually determine:

```text
Who is Alice?
       |
       v
What is Alice's SID?
       |
       v
Which group SIDs does Alice have?
       |
       v
What ACL protects the resource?
       |
       v
Do those SIDs have the required rights?
       |
       v
ALLOW / DENY
```

---

# 63. Share permissions vs NTFS permissions

For a Windows network share, there may be two layers.

Example:

```text
\\FILE01\HR
```

### Share permissions

Apply to the network share.

### NTFS permissions

Apply to the underlying filesystem.

So:

```text
Network access
    |
    +--> Share permissions
    |
    +--> NTFS permissions
```

The effective result depends on both.

---

# 64. Administrative shares

You've probably encountered things like:

```text
C$
ADMIN$
IPC$
```

These are special administrative shares.

For example:

```text
\\DC01\C$
```

provides access to the C: drive subject to the relevant administrative credentials and Windows security controls.

This is why seeing:

```text
\\dc01.warfare.corp\C$
```

in AD labs means you are interacting with an administrative share, not some special "AD protocol."

---

# 65. Trusts

Now we come to one of the most important AD concepts for multi-domain environments.

A:

> **Trust**

is a relationship allowing security principals in one domain/forest to be recognized across another security boundary according to the trust configuration.

Example:

```text
Domain A
    |
    | Trust
    |
Domain B
```

Trusts are not simply:

> "Domain A trusts Domain B, therefore everyone in A can access everything in B."

That is incorrect.

Authentication and authorization still matter.

---

# 66. Parent-child trust

Suppose:

```text
company.com
     |
     +-- india.company.com
```

AD can establish a relationship between parent and child domains.

These are normally **transitive** trust relationships within the forest.

---

# 67. Tree-root trust

Different trees in the same forest can be connected through trust relationships.

Example:

```text
Tree 1
company.com

Tree 2
research.org
```

Both can exist in one forest.

---

# 68. Forest trust

A forest can establish trust with another forest.

Example:

```text
Forest A
    |
    | Forest Trust
    |
Forest B
```

This is very important in mergers, acquisitions, partnerships, etc.

---

# 69. Trust direction

Trust direction can be confusing.

Think:

```text
A trusts B
```

This means the relationship is about which security principals can be authenticated/accepted through that trust path.

Trust direction does **not** automatically mean:

```text
B admins control A
```

or:

```text
Everyone in B can access A
```

Permissions still determine access.

---

# 70. Transitive vs non-transitive trust

### Transitive

Trust can extend through another trusted domain.

Example:

```text
A trusts B
B trusts C

=> possible trust path A -> C
```

depending on the trust architecture.

### Non-transitive

Only the directly established relationship applies.

---

# 71. SID History

Another very important AD security concept:

> **sIDHistory**

It allows an account to retain historical SID information, often during domain migrations.

Example:

```text
Old domain SID
       |
       v
SIDHistory on new account
       |
       v
Resource still recognizes old SID
```

This was designed to help migrations but can become security-sensitive because SID-based authorization is powerful.

---

# 72. AD delegation

Delegation is when one identity is allowed to act on behalf of another identity toward a service.

Kerberos delegation exists in multiple forms, including:

* Unconstrained delegation
* Constrained delegation
* Resource-based constrained delegation (RBCD)

These are advanced topics, but they are extremely important in AD security.

---

# 73. Unconstrained delegation

Conceptually:

```text
User
  |
  v
Server A
  |
  | can act on behalf of user
  v
Other services
```

This is powerful and historically considered risky because credentials/tickets can become exposed through highly privileged service contexts.

---

# 74. Constrained delegation

Instead of:

```text
Server A -> almost anything
```

the administrator can restrict delegation:

```text
Server A
   |
   +--> only Service B
   +--> only Service C
```

This reduces the scope.

---

# 75. Resource-Based Constrained Delegation

RBCD changes where much of the delegation decision is configured.

Rather than thinking:

```text
"Service account decides where it can delegate"
```

think:

```text
Target resource
       |
       v
defines which identity is allowed to act on its behalf
```

This appears frequently in modern AD attack-path analysis.

---

# 76. Common AD authentication/security attacks

Since you're studying cybersecurity, you should know these terms.

### Kerberoasting

Targets Kerberos service accounts through service-ticket-related mechanisms to attempt offline password cracking.

Central concept:

```text
SPN
 |
 v
Service account
 |
 v
Kerberos service ticket
 |
 v
Offline password cracking attempt
```

---

### AS-REP Roasting

Targets accounts configured so that certain Kerberos pre-authentication protections are absent.

Conceptually:

```text
User account
  |
  | pre-authentication disabled
  v
AS-REP material
  |
  v
Offline cracking
```

---

### Pass-the-Hash

Instead of using the user's plaintext password, an attacker may attempt to authenticate using a captured NTLM password hash where the protocol/authentication path permits it.

Concept:

```text
Password
   |
   v
NTLM hash
   |
   v
Authentication
```

---

### Pass-the-Ticket

Uses captured Kerberos tickets to impersonate the associated security context.

Conceptually:

```text
Kerberos ticket
      |
      v
Reuse ticket
      |
      v
Access service
```

---

# 77. Golden Ticket

This is one of the most famous AD attacks.

The core idea involves forging Kerberos TGTs using highly sensitive domain cryptographic material, historically associated with the **KRBTGT** account.

Conceptually:

```text
KRBTGT secret
      |
      v
Forged TGT
      |
      v
Potential domain-wide impersonation
```

This is extremely serious because the KRBTGT account is central to Kerberos ticket signing in the domain.

---

# 78. Silver Ticket

A Silver Ticket is conceptually different.

Instead of forging a TGT for broad domain authentication, the focus is on forging a **service ticket** for a particular service.

Think:

```text
Golden Ticket
    -> TGT level

Silver Ticket
    -> Service-ticket level
```

---

# 79. DCSync

Another major term:

> **DCSync**

This abuses directory replication functionality when an account has sufficiently powerful replication rights.

Conceptually:

```text
Attacker-controlled identity
        |
        | replication request
        v
Domain Controller
        |
        v
Sensitive directory credential material
```

This is why controlling high-privilege replication permissions is extremely dangerous.

---

# 80. KRBTGT account

Every AD domain has a special account:

```text
KRBTGT
```

Kerberos relies on cryptographic keys associated with this account for ticket operations.

You normally don't use KRBTGT as an ordinary user.

Its compromise has very serious domain-wide implications.

---

# 81. NTDS.dit

Remember:

```text
C:\Windows\NTDS\ntds.dit
```

This is the AD database on a domain controller.

Conceptually:

```text
DC01
 |
 +-- NTDS.dit
      |
      +-- Users
      +-- Groups
      +-- Computer accounts
      +-- Other directory data
```

Because of the sensitivity of the AD database, compromising a writable DC is a major security event.

---

# 82. Why a Domain Controller is so important

Think about an ordinary workstation:

```text
PC01
```

versus:

```text
DC01
```

Compromising PC01 may compromise:

```text
One machine
```

Compromising a DC can potentially expose:

```text
The domain
```

because a DC contains and controls extremely sensitive directory infrastructure.

This is why:

> **Domain Controller compromise is one of the most serious events in an AD environment.**

---

# 83. The Domain Controllers OU

By default, domain controllers are managed through a dedicated OU/container structure.

You might encounter:

```text
OU=Domain Controllers
```

Policies affecting DCs can be applied there.

---

# 84. Default groups you should know

Some common built-in groups include:

```text
Domain Admins
Domain Users
Enterprise Admins
Administrators
Backup Operators
Account Operators
Server Operators
DNS Admins
```

Their exact privileges differ.

A particularly important lesson:

> Group names are not the same thing as the complete effective security capability.

For example, membership in some non-obvious groups can provide very powerful indirect control.

---

# 85. Nested groups

Groups can contain other groups.

Example:

```text
Alice
   |
   v
HR Users
   |
   v
HR Managers
   |
   v
File Server Admins
```

This is one reason AD privilege analysis can become complicated.

Your effective permissions may come from:

* Direct group membership
* Nested groups
* Group nesting across domains
* Resource ACLs
* User rights
* GPOs
* Delegation

---

# 86. Security descriptors

Windows securable objects have security descriptors.

A security descriptor can include:

```text
Owner
DACL
SACL
```

The descriptor determines who can do what.

---

# 87. Common Windows rights

You may encounter permissions like:

```text
Read
Write
Modify
Delete
GenericRead
GenericWrite
GenericAll
WriteDACL
WriteOwner
```

Two especially important advanced concepts are:

### WriteDACL

Can potentially modify the object's DACL.

### WriteOwner

Can potentially change ownership, which may lead to further control depending on the situation.

These permissions often show up in AD attack-path analysis.

---

# 88. BloodHound and AD graphs

This brings us to a tool you're likely to encounter:

> **BloodHound**

BloodHound models relationships in AD as a graph.

Instead of thinking only:

```text
Alice -> Domain Admin
```

it can reveal chains such as:

```text
Low Priv User
     |
     v
Group membership
     |
     v
Write permissions
     |
     v
Computer
     |
     v
Admin access
     |
     v
Domain Admin path
```

The important concept is:

> **AD security is often about attack paths, not just individual permissions.**

---

# 89. Privilege escalation in AD

AD privilege escalation often looks like:

```text
Low-privileged user
        |
        v
Misconfiguration / weak permission
        |
        v
Higher privilege
        |
        v
Another relationship
        |
        v
Domain-level privilege
```

Possible causes include:

* Excessive group membership
* Dangerous ACLs
* Weak service-account passwords
* Kerberos weaknesses
* Delegation misconfiguration
* GPO abuse
* Local administrator reuse
* Credential exposure
* AD CS misconfiguration
* Trust misconfiguration

---

# 90. AD CS

Another advanced area:

> **Active Directory Certificate Services (AD CS)**

It allows organizations to deploy certificate-based identity and authentication infrastructure.

Conceptually:

```text
User / Computer
       |
       v
Certificate Authority
       |
       v
Certificate
       |
       v
Authentication / trust
```

Misconfigured certificate templates and enrollment permissions can create serious AD attack paths.

This is a major modern AD security topic.

---

# 91. Common AD protocols and ports

You should memorize the major ones.

| Protocol / Service       | Common Port |
| ------------------------ | ----------: |
| DNS                      |          53 |
| Kerberos                 |          88 |
| LDAP                     |         389 |
| LDAPS                    |         636 |
| SMB                      |         445 |
| RPC Endpoint Mapper      |         135 |
| Global Catalog           |        3268 |
| Global Catalog over TLS  |        3269 |
| Kerberos password change |         464 |
| NTP                      |         123 |

There are also many dynamically assigned RPC ports involved in Windows/AD communication.

---

# 92. SMB and AD

SMB is not Active Directory itself.

But AD environments rely heavily on SMB.

For example:

```text
\\DC01\SYSVOL
\\DC01\NETLOGON
```

and administrative shares:

```text
\\DC01\C$
```

SMB commonly uses:

```text
TCP 445
```

---

# 93. NETLOGON

You may see:

```text
NETLOGON
```

related to Windows domain operations.

The NETLOGON service participates in tasks involving:

* Domain authentication
* Secure channel operations
* DC discovery-related functionality
* Domain logon support

---

# 94. Time synchronization

This seems unrelated until you work with Kerberos.

Suppose:

```text
Client = 4:00 PM
DC = 4:20 PM
```

A sufficiently large clock difference can cause Kerberos authentication problems.

Therefore:

```text
NTP
 |
 v
Time synchronization
 |
 v
Kerberos reliability
```

Time in AD is a security dependency.

---

# 95. Service accounts

A service may need an identity.

For example:

```text
SQL Server
      |
      v
SQLService account
```

Instead of running as a normal human user.

Traditional service accounts can be risky if:

* Passwords are weak
* Passwords never change
* Excessive privileges are assigned

This is one reason Microsoft introduced:

> **gMSA = Group Managed Service Account**

where Windows can manage the service account password automatically.

---

# 96. gMSA

A Group Managed Service Account is designed to make service-account password management safer.

Conceptually:

```text
Service
   |
   v
gMSA
   |
   +--> Managed password
   +--> Automatic password changes
```

This reduces the need for administrators to hardcode long-lived passwords.

---

# 97. AdminSDHolder

Another advanced AD security concept is:

> **AdminSDHolder**

It helps protect highly privileged accounts/groups from unauthorized permission changes.

The associated process involves:

```text
AdminSDHolder
      |
      v
Protected objects
      |
      v
Security descriptor enforcement
```

You may encounter this during privilege-escalation investigations.

---

# 98. SDProp

Related to AdminSDHolder is:

> **SDProp = Security Descriptor Propagator**

It periodically helps ensure protected accounts/groups maintain appropriate security descriptors.

So:

```text
AdminSDHolder
      |
      v
SDProp
      |
      v
Protected privileged objects
```

---

# 99. LDAP vs Kerberos vs NTLM

This is one of the easiest places to get confused.

They serve different roles.

```text
LDAP
  =
Directory access

Kerberos
  =
Authentication

NTLM
  =
Legacy authentication
```

And:

```text
DNS
  =
Service/location discovery

SMB
  =
File/resource sharing
```

They can all work together in one AD environment.

---

# 100. One complete example

Let's combine everything.

Suppose a company has:

```text
Forest:
company.com

Domain:
corp.company.com

DCs:
DC01
DC02

OUs:
HR
IT
Finance

User:
Alice

Computer:
PC01

File server:
FILE01
```

The architecture could look like:

```text
                    FOREST
                       |
                company.com
                       |
                DOMAIN/TREE
                       |
              corp.company.com
                       |
       +---------------+---------------+
       |               |               |
      HR              IT            Finance
       |               |               |
     Alice            Bob            Sarah
       
                       |
                  Domain Controllers
                       |
                 +-----+-----+
                 |           |
                DC01        DC02
                 |
          +------+------+
          |             |
       Kerberos        LDAP
          |
          +---- DNS
          |
          +---- SYSVOL
```

Now Alice logs into PC01.

---

# 101. Alice logs in

```text
Alice
 |
 | username/password
 v
PC01
 |
 | DNS discovery
 v
DC01
 |
 | Kerberos
 v
KDC
 |
 | TGT
 v
Alice
```

---

# 102. Alice accesses FILE01

```text
Alice
 |
 | wants \\FILE01\HR
 v
Kerberos
 |
 | service ticket for FILE01
 v
FILE01
 |
 | checks access token
 v
Groups
 |
 | HR group
 v
ACL
 |
 v
ALLOW
```

Now you can see why AD is not one technology.

It is an ecosystem:

```text
DNS
 |
 v
DC discovery
 |
 v
Kerberos
 |
 v
Identity
 |
 v
Access Token
 |
 v
Groups
 |
 v
ACL
 |
 v
Resource
```

---

# 103. Why DNS, Kerberos and LDAP are all needed

Imagine:

### DNS disappears

Client may not know where the DC/services are.

### Kerberos disappears

Modern AD authentication breaks or falls back depending on the scenario.

### LDAP disappears

Applications cannot normally use the directory through LDAP.

### DC disappears

If other DCs exist, clients may still authenticate against them.

### SYSVOL breaks

Group Policy/logon script functionality can be affected.

This illustrates that AD is a collection of tightly connected services.

---

# 104. Forest vs domain vs OU — the easiest way to remember

Use this hierarchy:

```text
FOREST
  ↓
TREE
  ↓
DOMAIN
  ↓
OU
  ↓
OBJECT
```

Think of a university:

```text
Forest = Entire university system
Tree   = Academic branch
Domain = Campus
OU     = Department
Object = Student/teacher/computer
```

It's not a perfect analogy, but it's useful for remembering the hierarchy.

---

# 105. Security boundaries

This distinction is extremely important:

### Forest

Strongest traditional logical/security boundary.

### Domain

Administrative/security boundary within the forest, but trusts and forest-level relationships can connect domains.

### OU

Organizational/administrative container, **not a standalone security boundary**.

### Computer

Machine security boundary.

---

# 106. AD vs Azure AD / Microsoft Entra ID

You may see:

```text
Active Directory
```

and:

```text
Microsoft Entra ID
```

These are not identical products.

Traditional:

> **Active Directory Domain Services**

is heavily centered around:

```text
Domain
Kerberos
LDAP
Domain Controllers
Group Policy
Computer accounts
OUs
```

Microsoft Entra ID is Microsoft's cloud identity platform.

They can integrate, but don't treat them as the same thing.

---

# 107. The most important AD accounts/concepts to remember

For cybersecurity, these should be familiar:

```text
Administrator
Domain Admins
Enterprise Admins
KRBTGT
Guest
SYSTEM
Local Administrator
Computer Accounts
Service Accounts
gMSA
```

---

# 108. What happens when a user logs into a domain?

A simplified mental model:

```text
              USER LOGIN
                   |
                   v
            Find DC using DNS
                   |
                   v
              Kerberos
                   |
                   v
          Authentication succeeds
                   |
                   v
             Access Token
                   |
          +--------+--------+
          |                 |
          v                 v
       User SID         Group SIDs
          |                 |
          +--------+--------+
                   |
                   v
          Apply Group Policy
                   |
                   v
          User accesses resource
                   |
                   v
          ACL authorization
                   |
             +-----+-----+
             |           |
           ALLOW        DENY
```

That's one of the most useful diagrams to remember.

---

# 109. The cybersecurity mental model

When you enter an AD lab, don't think:

> "I need to find a password."

Think:

```text
IDENTITY
   ↓
AUTHENTICATION
   ↓
GROUP MEMBERSHIP
   ↓
PERMISSIONS
   ↓
TRUSTS
   ↓
DELEGATION
   ↓
PRIVILEGE
   ↓
DOMAIN / FOREST
```

An AD attack path often looks like:

```text
Initial Access
      ↓
User Account
      ↓
Group Membership
      ↓
ACL Misconfiguration
      ↓
Computer Control
      ↓
Credential/Token Access
      ↓
Higher Privilege
      ↓
Domain-Level Control
```

---

# 110. What you should study next

For cybersecurity specifically, I'd divide AD into these layers:

### Level 1 — Foundation

```text
Domain
DC
Forest
Tree
OU
Users
Computers
Groups
SID
RID
DNS
LDAP
```

### Level 2 — Authentication

```text
Kerberos
KDC
AS
TGS
TGT
SPN
NTLM
LSASS
Secure Channel
```

### Level 3 — Authorization

```text
Access Token
SID
Group membership
ACL
ACE
DACL
SACL
Share permissions
NTFS permissions
```

### Level 4 — Administration

```text
GPO
SYSVOL
GPC
GPT
Sites
Replication
Global Catalog
FSMO
```

### Level 5 — Trusts & advanced AD

```text
Trusts
SIDHistory
Delegation
Unconstrained Delegation
Constrained Delegation
RBCD
AD CS
gMSA
AdminSDHolder
SDProp
```

### Level 6 — AD security

```text
Kerberoasting
AS-REP Roasting
Password spraying
Pass-the-Hash
Pass-the-Ticket
Golden Ticket
Silver Ticket
DCSync
NTLM relay
ACL abuse
GPO abuse
AD CS abuse
Delegation abuse
```

---

# 111. The one-page mental map

Keep this picture in your head:

```text
                         ACTIVE DIRECTORY
                                |
             +------------------+------------------+
             |                  |                  |
          STRUCTURE         AUTHENTICATION     AUTHORIZATION
             |                  |                  |
     +-------+-------+      +----+----+       +----+----+
     |       |       |      |         |       |         |
  Forest   Domain    OU  Kerberos   NTLM    Groups     ACLs
     |       |       |      |         |       |         |
   Tree     DC     Objects  TGT/TGS   Hash   SIDs      ACE
                                      |
                                      SPN
             |
       +-----+------+
       |            |
      DNS         LDAP
       |
       +----------------------+
                              |
                           SERVICES
                              |
                +-------------+-------------+
                |             |             |
               SMB          GPO          SYSVOL
                |
             FILES

             ADVANCED
                |
      +---------+----------+
      |         |          |
    Trusts   Delegation  AD CS
      |
  SIDHistory

             SECURITY
                |
     +----------+----------+
     |          |          |
 Kerberoast  Pass-the-   DCSync
             Ticket
                |
          Golden/Silver
             Ticket
```

## The 10 things I would memorize first

```text
1. AD DS = Microsoft's enterprise directory service
2. DC = server hosting AD DS
3. Forest = top-level AD boundary
4. Domain = collection/security boundary of AD objects
5. OU = organization + Group Policy container
6. DNS = helps clients find AD services
7. Kerberos = primary AD authentication protocol
8. LDAP = directory access protocol
9. SID = Windows security identity
10. GPO = centralized Windows configuration
```

And the most important relationship to remember is:

```text
DNS
 ↓
Find Domain Controller
 ↓
Kerberos
 ↓
Authentication
 ↓
SID + Groups
 ↓
Access Token
 ↓
ACL
 ↓
Authorization
 ↓
Resource Access
```

That chain connects a huge portion of everything you'll encounter in an Active Directory lab.

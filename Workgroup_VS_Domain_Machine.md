# Difference Between Authentication in a Normal Windows Machine and a Domain-Joined Windows Machine
The main difference between a normal (workgroup) Windows machine and a domain-joined Windows machine is where the user's credentials are verified and who controls authentication.

* Normal Windows machine: The computer itself verifies the user's credentials using its local account database (SAM).

* Domain-joined Windows machine: A Domain Controller (DC) typically verifies domain users' credentials using Active Directory (AD), commonly through Kerberos.

Let's understand this step by step, especially from an Active Directory and cybersecurity perspective.

# 1. Normal Windows machine (Workgroup)

A normal Windows machine that is not joined to a domain uses local authentication.

``` mermaid
flowchart TD
    A["👤 User<br/>Enters local username and password"] --> B["💻 Local Windows Machine"]
    B --> C["LSASS<br/>Handles authentication"]
    C --> D[("SAM<br/>Security Account Manager<br/>Stores local account information and password hashes")]
    D --> E{"Are the credentials valid?"}
    E -->|Yes| F["✓ Success<br/>Access granted"]
    E -->|No| G["✗ Failure<br/>Access denied"]

    style A fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style B fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style C fill:#e0f2fe,stroke:#0284c7,color:#075985
    style D fill:#f1f5f9,stroke:#64748b,color:#1e293b
    style F fill:#dcfce7,stroke:#16a34a,color:#14532d
    style G fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
```
### How does it work?

Suppose your Windows laptop has a local account:

* Username: `kuldeep`

* Password: `YourPassword`

When you log in:

1. You enter your username and password.

2. Windows passes the authentication request to the Local Security Authority (LSA), with LSASS handling the authentication process.

3. Windows checks the local account information in the SAM database.

4. If the credentials are valid, Windows authenticates you and creates a logon session.

The SAM stores password hashes rather than plain-text passwords.

Important: The computer does not need to contact any Domain Controller or Active Directory server to authenticate a local account.


# 2. Domain-joined Windows machine

A domain-joined machine is connected to an organization's Active Directory (AD) domain. Instead of every computer independently managing all user accounts, a Domain Controller centrally manages domain accounts and verifies domain logins.

``` mermaid
flowchart TD
    A["👤 Domain User<br/>CORP\\kuldeep"] --> B["💻 Domain-Joined Windows Machine<br/>PC01"]
    B --> C["LSASS<br/>Kerberos Authentication Package"]
    C --> D["🌐 Contact Domain Controller"]
    D --> E["🖥️ Domain Controller<br/>DC01"]
    E --> F["Kerberos KDC<br/>Key Distribution Center"]
    F --> G[("Active Directory<br/>Domain Accounts, Password-Related Data,<br/>Groups and Policies")]
    G --> H{"Authentication successful?"}
    H -->|Yes| I["Issue Kerberos TGT"]
    I --> J["✓ Success<br/>Create Domain Logon Session"]
    H -->|No| K["✗ Failure<br/>Logon Denied"]

    style A fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style B fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style C fill:#e0f2fe,stroke:#0284c7,color:#075985
    style D fill:#f1f5f9,stroke:#64748b,color:#1e293b
    style E fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style F fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style G fill:#f1f5f9,stroke:#64748b,color:#1e293b
    style I fill:#dcfce7,stroke:#16a34a,color:#14532d
    style J fill:#dcfce7,stroke:#16a34a,color:#14532d
    style K fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
```

### How does it work?

Suppose a company has an Active Directory domain:

* Domain: `corp.local`

* Domain Controller: `DC01`

* Domain user: `kuldeep`

* Computer: `PC01`

When you log in using `CORP\kuldeep`:

1. You enter your domain username and password on `PC01`.

2. Windows uses LSASS to process the authentication request.

3. The computer communicates with a Domain Controller.

4. The DC's Kerberos Key Distribution Center validates the user's authentication using Active Directory information.

5. If authentication succeeds, the DC issues Kerberos tickets, beginning with a Ticket Granting Ticket (TGT).

6. Windows establishes a logon session, and the user can access resources according to their permissions.

The Domain Controller also provides centralized account management, password policies, group memberships and other domain-related controls.

Important: Kerberos is the default authentication protocol in a typical modern Active Directory domain, but Windows can also use NTLM in certain circumstances, such as when Kerberos cannot be used.


# 3. Key differences

| Feature                    | Normal Windows (Workgroup)      | Domain-joined Windows                           |
| -------------------------- | ------------------------------- | ----------------------------------------------- |
| Authentication             | Local authentication            | Usually domain authentication                   |
| Account database           | Local SAM                       | Active Directory on DC                          |
| Verification               | Local computer                  | Domain Controller, when available               |
| Common protocols           | NTLM-based local authentication | Kerberos, with NTLM fallback                    |
| Account management         | Individually on each computer   | Centrally managed through AD                    |
| Password policies          | Configured locally              | Can be centrally managed using Group Policy     |
| Centralized login          | No                              | Yes                                             |
| Offline login              | Yes                             | Yes, for local accounts                         |
| Cached domain login        | Not applicable                  | Possible, if previously configured and cached   |
| Single sign-on (SSO)       | Limited                         | Supported through Kerberos and other mechanisms |
| Centralized access control | Limited                         | Supported through domain groups and policies    |

# 4. What happens if the Domain Controller is offline?

This is an important concept in Active Directory.

Suppose you have a domain-joined laptop and you previously logged in successfully using your domain account.

Now imagine that you take your laptop home, where it cannot connect to the organization's Domain Controller.

You may still be able to log in using cached domain credentials.

```mermaid
flowchart TD
    A["💻 Domain-Joined Laptop"] --> B["User Enters Domain Credentials"]
    B --> C{"Can the laptop contact a DC?"}
    C -->|Yes| D["Authenticate with Domain Controller"]
    D --> E["✓ Domain Logon Successful"]
    C -->|No| F{"Are cached domain credentials available?"}
    F -->|Yes| G["Verify Against Cached Logon Information"]
    G --> H{"Credentials match?"}
    H -->|Yes| I["✓ Offline Domain Logon Successful"]
    H -->|No| J["✗ Logon Denied"]
    F -->|No| K["✗ Domain Logon Unavailable"]

    style A fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style D fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style E fill:#dcfce7,stroke:#16a34a,color:#14532d
    style I fill:#dcfce7,stroke:#16a34a,color:#14532d
    style J fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    style K fill:#fee2e2,stroke:#dc2626,color:#7f1d1d

```

Cached logon does not mean the laptop has become a local account. It is still a domain account, and the offline authentication is based on previously cached verification data.

# 5. Does a domain-joined machine still have a SAM database?

Yes! This is a common point of confusion.

Even after joining a Windows computer to a domain, the computer retains its local SAM database.

For example, a domain-joined computer may have:

* Local account: `PC01\administrator`

* Domain account: `CORP\kuldeep`

Both accounts can be used to log in, provided they are enabled and permitted to log on.
```mermaid
flowchart TD
    A["💻 Domain-Joined Computer"] --> B["Local Account Authentication"]
    A --> C["Domain Account Authentication"]

    B --> D[("Local SAM")]
    D --> E["PC01\\administrator"]
    E --> F["Authenticated Locally"]

    C --> G["Domain Controller"]
    G --> H[("Active Directory")]
    H --> I["CORP\\kuldeep"]
    I --> J["Authenticated Through Domain"]

    style A fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style B fill:#e0f2fe,stroke:#0284c7,color:#075985
    style C fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style D fill:#f1f5f9,stroke:#64748b,color:#1e293b
    style H fill:#f1f5f9,stroke:#64748b,color:#1e293b
    style F fill:#dcfce7,stroke:#16a34a,color:#14532d
    style J fill:#dcfce7,stroke:#16a34a,color:#14532d
```

You can explicitly specify which account you want to use on the Windows login screen:

* `.\administrator` — the computer's local account.

* `CORP\kuldeep` — a domain account.

* `kuldeep@corp.local` — a domain account using its User Principal Name (UPN), assuming that is the account's UPN.

# 6. Authentication vs authorization

These two concepts are especially important when learning Active Directory exploitation.

* Authentication: Proves who you are.

* Authorization: Determines what you are allowed to access.

For example, a domain user might successfully authenticate to a computer but not have permission to install software or access a restricted folder.

Active Directory group memberships, local groups, file permissions, security policies and other access-control mechanisms can determine what that user is authorized to do.

```mermaid
flowchart TD
    A["👤 User"] --> B["Authentication<br/>Who are you?"]
    B --> C{"Credentials Valid?"}
    C -->|No| D["Access Denied"]
    C -->|Yes| E["Authenticated"]
    E --> F["Authorization<br/>What can you access?"]
    F --> G["Check Groups, Permissions<br/>and Security Policies"]
    G --> H{"Permission Granted?"}
    H -->|Yes| I["✓ Access to Resource"]
    H -->|No| J["✗ Access Restricted"]

    style B fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style F fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style I fill:#dcfce7,stroke:#16a34a,color:#14532d
    style D fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    style J fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
```

# 7. How this relates to Kerberos and NTLM

Kerberos

* The default authentication protocol in modern Active Directory domains.

* Uses tickets, such as TGTs and service tickets.

* Supports mutual authentication and single sign-on.

* Uses a Key Distribution Center hosted on the Domain Controller.

NTLM

* A challenge-response authentication protocol.

* Can be used for local accounts and in certain domain authentication scenarios.

* Does not use Kerberos-style tickets.

* May be used when Kerberos is unavailable or incompatible with a particular authentication scenario.

```mermaid
flowchart TD
    A["Windows Authentication"] --> B["Local Account"]
    A --> C["Domain Account"]

    B --> D["Local Authentication"]
    D --> E["Local SAM"]
    E --> F["NTLM-based Authentication"]

    C --> G{"Authentication Protocol"}
    G --> H["Kerberos"]
    G --> I["NTLM Fallback"]

    H --> J["Domain Controller KDC"]
    J --> K["TGT and Service Tickets"]
    I --> L["Challenge-Response"]
    L --> M["Domain Controller Verification"]

    style A fill:#dbeafe,stroke:#2563eb,color:#1e40af
    style H fill:#ccfbf1,stroke:#0d9488,color:#134e4a
    style I fill:#e0f2fe,stroke:#0284c7,color:#075985
    style K fill:#dcfce7,stroke:#16a34a,color:#14532d
    style M fill:#dcfce7,stroke:#16a34a,color:#14532d

```

From a cybersecurity perspective, Kerberos tickets, NTLM authentication, cached credentials and the distinction between local and domain accounts are all important concepts in Windows security and Active Directory assessments.

# 8. Simple Analogy

A normal Windows machine is like a building where each building manager maintains their own list of authorized people.

A domain-joined Windows machine is like a company where a central office manages employee identities and provides authentication services to multiple buildings.

Each building can still have its own local accounts and access rules, even though it belongs to the same company.

# Key takeaway:

• **Workgroup**: Each computer manages its own local accounts and authentication.

• **Domain**: Active Directory centrally manages domain accounts, and Domain Controllers provide authentication services.

• **Domain-joined computers**: Still retain local accounts and a SAM database, and can support offline domain logons through cached credentials.
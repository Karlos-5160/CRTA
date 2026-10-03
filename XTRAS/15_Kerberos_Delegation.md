# Kerberos Delegation

Kerberos Delegation in Active Directory is a mechanism that allows a service to access another service on behalf of a user, without asking the user to authenticate again.

Let's understand it with a simple example.

## 1. Imagine this scenario

Suppose you work in a company that has three systems:

* User (Kuldeep): You log in to your company computer.

* Web Server: Hosts the company's website.

* Database Server: Stores employee information.

Kuldeep (User)

Requests employee information

Web Server

Processes the user's request

Database Server

Contains employee records

You log in to the website and request your employee details.

The web server needs to retrieve those details from the database server.

But here's the problem:

How does the web server access the database on your behalf?

The web server cannot simply reuse your password, and it shouldn't have unrestricted access to every user's data.

This is where Kerberos Delegation comes in.


## 2. How Kerberos Delegation works

First, remember that Kerberos uses tickets instead of sending your password to every server.

1. You log in to the domain

   You authenticate to Active Directory, and the Key Distribution Center (KDC) issues you a Ticket Granting Ticket (TGT).

2. You access the Web Server

   You request access to the website. Your computer obtains a Kerberos service ticket for the Web Server and presents it to the server.

3. The Web Server needs the Database

   The Web Server needs to make another request to the Database Server on your behalf. This is known as the second hop.

4. Delegation allows the second hop

   If the account and services are configured for delegation, the Web Server can obtain an appropriate Kerberos ticket for the Database Server on your behalf.

5. The Database Server responds

   The Database Server validates the ticket and checks your authorized access. It then returns the permitted information to the Web Server.

The main idea

The Web Server accesses the Database Server using Kerberos delegation, while the database can identify the user on whose behalf the request is made. The user's password is never handed to the Web Server.

## 3. Why is Kerberos Delegation needed?

Without delegation, a common issue in Windows authentication is called the Double Hop Problem.

Without delegation

User

Web Server

First hop works

Database Server

Second hop may fail

The Web Server cannot automatically forward the user's original authentication to the Database Server. Without appropriate delegation or another authentication method, the second request may fail.

Delegation solves this problem by allowing the Web Server to obtain the necessary credentials or tickets for the second service under an approved configuration.

## 4. Types of Kerberos Delegation

There are three main types you should know in Active Directory.

1. Unconstrained Delegation

The front-end service can delegate a user's credentials to other services without being restricted to a specific list of destination services.

* Broad delegation capability.

* A compromise of a server configured this way can put delegated credentials at risk.

* Generally avoided in modern environments.

2. Constrained Delegation (KCD)

The administrator specifies which backend services the front-end service is allowed to access on behalf of users.

For example, a Web Server might be allowed to delegate only to a particular SQL Database service.

* Limits the permitted destination services.

* Can use Kerberos-only delegation or protocol transition, depending on its configuration.

3. Resource-Based Constrained Delegation (RBCD)

Instead of the front-end service's administrator specifying the permitted backend services, the resource (such as the Database Server) specifies which accounts or services are trusted to delegate to it.

* The destination resource controls who can delegate to it.

* Often useful in environments where different teams manage front-end and backend services.

## 5. Quick comparison

| Feature                     | Unconstrained                   | Constrained                              | Resource-based constrained                                 |
| --------------------------- | ------------------------------- | ---------------------------------------- | ---------------------------------------------------------- |
| Who defines the delegation? | Front-end account configuration | Front-end account configuration          | Target resource                                            |
| Destination restriction     | No specific service restriction | Specific allowed services                | Specific trusted front-end principals                      |
| Scope                       | Broad                           | Limited                                  | Limited                                                    |
| Security concern            | High exposure if compromised    | Depends on configuration and permissions | Depends on who can modify the target's delegation settings |

## 6. An easy analogy to remember

Imagine you're working in an office.

* You (user): An employee who needs access to a confidential file.

* Web Server: Your office assistant.

* Database Server: The records department.

* Kerberos: The office's identity verification and authorization ticket system.

* Delegation: Permission for your assistant to request specific records from the records department on your behalf.

With unconstrained delegation, the assistant has broad delegation capabilities. With constrained delegation, the assistant can request only specifically approved services. With resource-based constrained delegation, the records department decides which assistants it trusts to make requests.

Important for Active Directory security

Kerberos delegation is a legitimate feature used for multi-tier applications, but misconfigured delegation can create security risks. When reviewing an AD environment, administrators should identify which accounts are trusted for delegation, which services are permitted, and who has permission to change those configurations.

Remember for exams and interviews: Kerberos Delegation allows a service to access another service on behalf of a user without asking for the user's password again. Its three main types are unconstrained delegation, constrained delegation and resource-based constrained delegation (RBCD).

# Basion Host 

A Bastion Host is a specially secured computer or server that acts as a controlled entry point into a private network. It allows administrators to securely access internal servers without exposing those servers directly to the internet.

Think of it as a security checkpoint or guarded gate between the internet and your internal network.

## 1. Where does a Bastion Host sit: DMZ or MZ?

A bastion host is commonly placed in a DMZ (Demilitarized Zone), but it can also be placed in a dedicated management zone (MZ), depending on the network architecture.

![05 Firewalls - HackMD](https://images.openai.com/static-rsc-4/1ue72z5cAIVM93O6_AGTBvZRMz4c4MD5ppXIaW9lFmrNEHUZ-1N9zA2NCwltbMjxkBgismWjQafbTLYwnwaLTBIwA6k68chBJJDw5C6_7Zr0oWUY65zM0QpvbQ-0PV4dzY77KABEEQycny5-F_27VK8IPfWy0ZzKCjfKcNwj574?purpose=inline)

Conceptual network layout: the bastion host is placed in a controlled zone between external access and internal resources.

* DMZ: A separate network segment between the internet and the internal network, often used for systems that need controlled external access.

* MZ (Management Zone): A dedicated network segment for managing infrastructure, such as servers, network devices and security appliances.

* Bastion host: The hardened machine that administrators connect to before accessing protected internal systems.

The exact placement depends on the organization's design. For example, a bastion host exposed to administrator connections from the internet may be in a DMZ, while an internal-only bastion host may be in a management zone.


## 2. How does a Bastion Host work?

Imagine a company has three internal servers that should not be directly accessible from the internet.

Administrator's PC

Remote administrator

SSH / RDP + MFA

Bastion Host

Hardened entry point

DMZ / Management Zone

Controlled, authenticated access

Private Internal Network

Web Server

Database

File Server

Here's the process:

1. An administrator wants to manage an internal server from their laptop.

2. They connect to the bastion host, typically using SSH or RDP, with strong authentication such as MFA.

3. After authentication, the bastion host allows them to access only the internal systems permitted by firewall rules and their account privileges.

4. The administrator manages the internal server through this controlled connection.

5. Access can be logged and monitored for auditing and investigation.

The internal servers do not need to accept direct connections from the internet.

## 3. Why do we use a Bastion Host?

| Purpose                | Explanation                                                                              |
| ---------------------- | ---------------------------------------------------------------------------------------- |
| Reduced attack surface | Internal servers don't need to expose management ports to the internet.                  |
| Centralized access     | Administrators can access multiple permitted servers through one controlled entry point. |
| Auditing               | Connections and administrative activity can be logged and monitored.                     |
| Access control         | Only authorized users can connect, with permissions restricted by role.                  |
| Network segmentation   | Helps isolate the internal network from direct external access.                          |

A bastion host itself is a high-value target. That's why it should be hardened, regularly patched, protected with MFA and monitored. It does not automatically make the internal network secure.

## 4. Is a Bastion Host only used in cloud networks?

No. Bastion hosts are used in both traditional on-premises networks and cloud networks.

![Dataviztech | PI System & Data Visualization Experts](https://images.openai.com/static-rsc-4/g-DJWkqbRmdgLdJZLixNkZV1v8ONwDl6065vvb_ztICTLc2Fn1otPBlUejQWGyPPfps4Jbhf-kukZbtPNB_dTZ1xA-XymGTx56JA7W_xqL-7xtyAf1cfe3hoYjXTnc8HiKg9BcVGin2q3x71St_GfRHHiWoLXZNlWKNVHWKh7iM?purpose=inline)

On-premises networks

An organization has physical servers in its own data center. It deploys a bastion host in a DMZ or management network so administrators can securely access internal servers remotely.

![Alkamys IT Consulting](https://images.openai.com/static-rsc-4/iOfEpm9gkyYy7wHDfC4BHkrGAFxkdt75j41wZx_Bq0ebyjY_7unao8FS_dKetw3YBMsCjbhqQq0peaVIZR5-wmV2ISiBDbbT8_JRbi4fdl_FTFh5J5eZzeRlmx-Rc4Wj3oFOXe7EF_bvqn8FLNhz5vlVvYvU2zKlmJJGOc2rpFo?purpose=inline)

Cloud networks (AWS, Azure, GCP)

An organization deploys a bastion host in a public subnet or dedicated access subnet to provide controlled SSH or RDP access to virtual machines in private subnets.

![Zenflux - Enterprise Observability Platform](https://images.openai.com/static-rsc-4/4jWqvtcbbqhJcMU2FMpCQMjr_V2LgohvpY5W-VtmATJDl5Y-l0SpwXpeIy40QqOKiZUDackMnASH75Ncttavlvh58XUwGjxgHb-U5Qshys3vvqKg8ufpT6Z0YJFDOgyo_nwAmfMWzZIRdoIKYou-Gy7bfSV_pHeueU6213yZP7A?purpose=inline)

Hybrid networks

An organization connects its on-premises data center to a cloud environment. Bastion hosts or other controlled access systems can be used in either environment, depending on the design.

## 5. Bastion Host vs. Jump Server

These terms are often used interchangeably, although there can be a distinction.

| Bastion Host                                                            | Jump Server                                                     |
| ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| A hardened system designed to withstand exposure to untrusted networks. | An intermediary server used to access another system.           |
| Often placed in a DMZ or dedicated access zone.                         | Can be placed in a DMZ, management network or internal network. |
| Focuses on secure, controlled access.                                   | Focuses on providing a connection path between networks.        |

A bastion host is often a type of jump server, but not every jump server is necessarily hardened to the same standard as a bastion host.

## 6. Bastion Host in cybersecurity labs

For example, imagine you are managing a lab with a Kali Linux machine, a bastion host and an internal Windows server.

* Kali Linux is your administrator workstation.

* The bastion host is the authorized gateway.

* The Windows server is on a private network and does not accept direct external management connections.

You connect to the bastion host first and then use the permitted management protocol to connect to the Windows server.

This is also why bastion hosts are relevant to network security, firewall configuration, segmentation and access auditing.

A bastion host is a hardened system that provides controlled access to a protected network. It is commonly located in a DMZ or management zone and is used in on-premises, cloud and hybrid environments. Its purpose is to reduce direct exposure of internal systems and centralize secure administrative access.




# Is Every Server in the DMZ network is a basion host ?

No. Not every server in a DMZ is a bastion host. A bastion host is a specific type of hardened server designed to provide controlled access to other systems. A DMZ can contain several different types of servers, each serving a different purpose.

## 1. Different servers in a DMZ

DMZ NETWORK

Web Server

Hosts a website accessible from the internet

Bastion Host

Controlled administrative access to internal systems

Mail Gateway

Filters and relays email traffic

Reverse Proxy

Forwards incoming requests to backend servers

| Server        | Main purpose                                                                 |
| ------------- | ---------------------------------------------------------------------------- |
| Web server    | Serves websites and web applications to users.                               |
| Bastion host  | Provides a controlled gateway for administrative access to internal systems. |
| Mail gateway  | Filters and relays email between networks.                                   |
| Reverse proxy | Receives client requests and forwards them to appropriate backend servers.   |
| DNS server    | May provide publicly accessible authoritative DNS services.                  |
| VPN gateway   | Provides authenticated remote access to a private network.                   |

## 2. The main difference

Internet-facing web server

Its purpose is to serve content to external users. For example, when you visit `example.com`, your browser sends a request to the web server.

Bastion host

Its purpose is to let authorized administrators securely access internal systems, such as private web servers, databases or Windows machines.

Both may be hardened and located in the DMZ, but their roles are different.

## 3. An important concept

A DMZ is a network zone, not a type of server.

For example, an organization might have:

* A DMZ containing a public web server and a bastion host.

* An internal network containing database servers and application servers.

* A management zone containing monitoring and administration systems.

Also, not every internet-facing server has to be in a DMZ; the network architecture determines its placement.

Remember: A bastion host is defined by its security role as a controlled access point, not simply by being located in a DMZ.

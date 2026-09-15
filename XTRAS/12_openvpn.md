# When you connect via OpenVPN, traffic flows back to your machine using **assigned virtual IP addresses**, **routing table rules**, and **packet encapsulation**.


### 1. The Virtual IP Assignment

When your OpenVPN client connects to the lab's VPN server:

* OpenVPN creates a virtual interface on your machine (usually `tun0`).
* The lab's OpenVPN server assigns your `tun0` interface a private IP address within a dedicated VPN pool (for example, `10.8.0.2`), while the VPN server itself takes `10.8.0.1`.

---

### 2. Outgoing Traffic (You ➔ Target Host)

1. You run a command targeting a lab machine (e.g., `192.168.98.15`).
2. Your operating system checks its routing table (which OpenVPN populated upon connecting) and sees that `192.168.98.0/24` routes through `tun0`.
3. OpenVPN grabs the packet, encrypts it, and sends it across the internet to the lab VPN server's public IP.
4. The lab VPN server decrypts the packet. At this stage, the packet looks like:
* **Source IP:** `10.8.0.2` (your `tun0` IP)
* **Destination IP:** `192.168.98.15` (target machine)


5. The VPN server forwards this packet onto the lab's local network (LAN).

---

### 3. Return Traffic (Target Host ➔ You)

How the target host knows where to send the reply depends on how the lab network is configured. It happens in one of two ways:

#### Case A: Routed Network (Standard Enterprise / Advanced Labs)

* The target machine (`192.168.98.15`) prepares a response addressed to destination `10.8.0.2`.
* Because `10.8.0.2` is not on its local subnet, the target hands the packet to its **default gateway** (which is either the VPN server itself or a core lab router).
* The lab router has a static route: `10.8.0.0/24 via 10.8.0.1` (the OpenVPN server).
* The VPN server receives the packet, recognizes `10.8.0.2` belongs to your active client session, encrypts it, and pushes it back over the public internet to your home IP.
* Your Kali machine decrypts it and hands it to `tun0`, completing the loop.

#### Case B: NAT / Masquerade (Typical Practice Labs like HTB / THM)

* When the lab VPN server decrypts your outbound packet in Step 2, it performs **NAT (Network Address Translation)** or `MASQUERADE` via `iptables`.
* It replaces the source address `10.8.0.2` with the **VPN server's own local LAN IP** (e.g., `192.168.98.1`).
* The target host sees the request coming from `192.168.98.1` (a system directly inside its local subnet) and replies straight back to `192.168.98.1`.
* The VPN server looks at its internal state translation table, maps the reply back to your session (`10.8.0.2`), encrypts the packet, and transmits it to your physical machine.

---

### Why Reverse Shells Work

Because your `tun0` IP (`10.8.0.2`) is explicitly routed by the VPN server:

* If you catch a reverse shell, you set your LHOST to your `tun0` address.
* When the target connects to `10.8.0.2`, the lab's routing rules or gateway know that all `10.8.0.0/24` traffic must be directed to the OpenVPN daemon, which delivers it straight to your listening Netcat/Metasploit port on `tun0`.
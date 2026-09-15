# Ligolo-ng turns your compromised machine into a VPN gateway rather than just a proxy, allowing tools on Kali to interact with the internal network as if your machine were physically plugged into it.

### The Two Core Components

* **`ligolo-proxy` (Attacker / Kali):**
Acts as the control server and VPN endpoint. It manages connected agents, provides the interactive CLI, and attaches directly to your virtual `ligolo` TUN interface to read and write raw IP packets.
* **`agent` (Pivot / Target):**
A lightweight client dropped on the compromised host (`192.168.80.10`). It reaches out to Kali over an encrypted TLS connection (`:11601`), handles traffic translation, and acts as the bridge into the internal subnet (`192.168.98.0/24`).

---

### End-to-End Packet Flow

When you run a command targeting an internal Active Directory IP (for example, `curl http://192.168.98.15`):

```text
[Tool: curl]
    │
    ▼
[Kali Kernel Routing Table]
    │  (Matches rule: 192.168.98.0/24 dev ligolo)
    ▼
[Virtual TUN Interface: ligolo]
    │  (Intercepts raw Layer 3 IP packet)
    ▼
[ligolo-proxy process]
    │  (Encapsulates packet into TLS stream)
    ▼  === TLS Tunnel over port 11601 ===
[ligolo-agent (Pivot 192.168.80.10)]
    │  (De-encapsulates packet via gVisor userland TCP stack)
    ▼
[Internal Subnet 192.168.98.0/24] ──► [Target AD Host 192.168.98.15]

```

1. **Tool Invocation:** `curl` creates an outgoing TCP packet with destination `192.168.98.15:80`.
2. **Kernel Route Matching:** The Linux kernel looks up `192.168.98.15` in its routing table. Because you ran `ip route add 192.168.98.0/24 dev ligolo`, the kernel knows not to send it to the local Wi-Fi/Ethernet gateway; instead, it hands the raw IP packet directly to the `ligolo` virtual device.
3. **Capture by Proxy:** The `ligolo-proxy` process holds an open file descriptor on `/dev/net/tun` for the `ligolo` device. It reads the raw IP packet, wraps it in the established TLS connection, and transmits it over the network to the agent.
4. **Translation by Agent:** The agent on `192.168.80.10` receives the payload. Ligolo-ng embeds **gVisor's netstack** (a userland TCP/IP stack written in Go). Because of this:
* The agent parses the incoming IP packet in user space.
* It translates the request into an outbound connection from the pivot machine to `192.168.98.15:80`.
* **Crucial benefit:** Because it creates regular user-space connections on the pivot, the agent does **not** require root privileges or kernel module access on the victim machine.


5. **Target Receives Traffic:** The target host (`192.168.98.15`) sees a normal TCP connection arriving from the pivot host's internal interface.
6. **Return Path:** When the AD host sends a reply back to the pivot, the agent captures the response bytes, sends them back through the TLS tunnel to `ligolo-proxy`, and the proxy injects the response packet back into the `ligolo` TUN device. The Kali kernel marks it as an incoming response and hands it to `curl`.

---

### Why This Differs from SSH SOCKS + Proxychains

* **Proxychains (Layer 7 Hooking):** Forces standard user-space tools to talk through a SOCKS proxy by intercepting dynamic C library calls (`LD_PRELOAD`). It does not work with statically compiled binaries, Go binaries, raw sockets, or tools that ignore libc wrappers.
* **Ligolo-ng (Layer 3 Routing):** Routes IP packets directly at the kernel network level. Any standard tool (`nmap`, `crackmapexec`/`netexec`, `impacket`, `smbclient`, `bloodhound`, or a web browser) functions natively against internal IPs without modification.Ligolo-ng turns your compromised machine into a VPN gateway rather than just a proxy, allowing tools on Kali to interact with the internal network as if your machine were physically plugged into it.

---

### The Two Core Components

* **`ligolo-proxy` (Attacker / Kali):**
Acts as the control server and VPN endpoint. It manages connected agents, provides the interactive CLI, and attaches directly to your virtual `ligolo` TUN interface to read and write raw IP packets.
* **`agent` (Pivot / Target):**
A lightweight client dropped on the compromised host (`192.168.80.10`). It reaches out to Kali over an encrypted TLS connection (`:11601`), handles traffic translation, and acts as the bridge into the internal subnet (`192.168.98.0/24`).

---

### End-to-End Packet Flow

When you run a command targeting an internal Active Directory IP (for example, `curl [http://192.168.98.15](http://192.168.98.15)`):

```text
[Tool: curl]
    │
    ▼
[Kali Kernel Routing Table]
    │  (Matches rule: 192.168.98.0/24 dev ligolo)
    ▼
[Virtual TUN Interface: ligolo]
    │  (Intercepts raw Layer 3 IP packet)
    ▼
[ligolo-proxy process]
    │  (Encapsulates packet into TLS stream)
    ▼  === TLS Tunnel over port 11601 ===
[ligolo-agent (Pivot 192.168.80.10)]
    │  (De-encapsulates packet via gVisor userland TCP stack)
    ▼
[Internal Subnet 192.168.98.0/24] ──► [Target AD Host 192.168.98.15]

```

1. **Tool Invocation:** `curl` creates an outgoing TCP packet with destination `192.168.98.15:80`.
2. **Kernel Route Matching:** The Linux kernel looks up `192.168.98.15` in its routing table. Because you ran `ip route add 192.168.98.0/24 dev ligolo`, the kernel knows not to send it to the local Wi-Fi/Ethernet gateway; instead, it hands the raw IP packet directly to the `ligolo` virtual device.
3. **Capture by Proxy:** The `ligolo-proxy` process holds an open file descriptor on `/dev/net/tun` for the `ligolo` device. It reads the raw IP packet, wraps it in the established TLS connection, and transmits it over the network to the agent.
4. **Translation by Agent:** The agent on `192.168.80.10` receives the payload. Ligolo-ng embeds **gVisor's netstack** (a userland TCP/IP stack written in Go). Because of this:
* The agent parses the incoming IP packet in user space.
* It translates the request into an outbound connection from the pivot machine to `192.168.98.15:80`.
* **Crucial benefit:** Because it creates regular user-space connections on the pivot, the agent does **not** require root privileges or kernel module access on the victim machine.


5. **Target Receives Traffic:** The target host (`192.168.98.15`) sees a normal TCP connection arriving from the pivot host's internal interface.
6. **Return Path:** When the AD host sends a reply back to the pivot, the agent captures the response bytes, sends them back through the TLS tunnel to `ligolo-proxy`, and the proxy injects the response packet back into the `ligolo` TUN device. The Kali kernel marks it as an incoming response and hands it to `curl`.

---

### Why This Differs from SSH SOCKS + Proxychains

* **Proxychains (Layer 7 Hooking):** Forces standard user-space tools to talk through a SOCKS proxy by intercepting dynamic C library calls (`LD_PRELOAD`). It does not work with statically compiled binaries, Go binaries, raw sockets, or tools that ignore libc wrappers.
* **Ligolo-ng (Layer 3 Routing):** Routes IP packets directly at the kernel network level. Any standard tool (`nmap`, `crackmapexec`/`netexec`, `impacket`, `smbclient`, `bloodhound`, or a web browser) functions natively against internal IPs without modification.
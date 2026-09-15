# when i was doing a nmap scan with normal [nmap -sn 192.168.98.0/24] i was getting all 256 hosts as up but when i did same nmap scan with --unprivileged flag i got only 4 hosts to be up why ?

The discrepancy happens because running Nmap as root (privileged) versus unprivileged completely changes the underlying network packets sent to probe hosts, and virtual TUN interfaces handle those packets differently.

---

### 1. Privileged Mode (`nmap -sn`)

When run as root without `--unprivileged`, Nmap crafts raw Layer 2/3 packets:

* **Direct Subnet Fallback (ARP):** If Nmap believes a target is directly on the local link, it defaults to sending ARP requests. Some tunnel/TUN devices or intermediate route configurations will respond to or absorb ARP/raw packets locally, leading Nmap to interpret every synthesized ARP reply as a valid live host.
* **Raw ICMP & TCP Probes:** Root Nmap sends an ICMP Echo request, a TCP SYN to port 443, a TCP ACK to port 80, and an ICMP timestamp request. If the virtual interface or intermediate stack returns resets or echoes across the route, Nmap flags all 256 addresses as alive.

---

### 2. Unprivileged Mode (`--unprivileged`)

When you force unprivileged mode, Nmap loses the ability to open raw sockets (`SOCK_RAW`) or send custom frame-level packets. It is forced to rely strictly on the standard operating system network stack via standard system calls:

* **TCP `connect()` Probes:** Instead of raw packets, Nmap issues standard TCP `connect()` calls (specifically targeting port 80 and port 443) using the kernel's normal routing table.
* **Accurate Handshake Handling:** A host is only marked "up" if a true TCP `SYN-ACK` is returned through the tunnel to complete the handshake, or an explicit `RST` is returned by an actual running host on that subnet.
* Because only genuine machines respond to the handshake through the Ligolo tunnel, the phantom results disappear, correctly narrowing the list down to the **4 real hosts**.

---

### Recommended Host Discovery over Ligolo

Because raw socket operations over virtual TUN tunnels frequently produce false positives, standard TCP-based probes are the most reliable:

* **Target High-Probability AD Ports:**
```bash
nmap -sT -Pn -p 88,135,389,445,3389 --open -T4 192.168.98.0/24

```


* `-sT`: Enforces standard TCP connect calls through the OS routing table.
* `-Pn`: Skips the ping phase so hosts blocking ICMP are still evaluated.
* `--open`: Displays only responsive nodes with open services.
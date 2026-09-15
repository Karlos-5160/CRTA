# I have a ip range 192.168.98.0/24 how to check for open hosts ? 

Scanning an entire `/24` subnet through Proxychains via an SSH SOCKS proxy requires specific handling because Proxychains creates a full TCP connection for every single attempt, which is slow and drops packets if swamped.

Here are the standard approaches to identify active hosts:

### 1. Nmap TCP Connect Scan (via Proxychains)

To scan through Proxychains, you must disable host discovery (`-Pn`), force full TCP connect mode (`-sT`), and target a few common ports. Avoid full port ranges or raw packet scans.

```bash
proxychains nmap -sT -Pn -p 22,80,445,3389 --open -T4 192.168.98.0/24

```

* **`-sT`**: Forces standard TCP handshake (required for SOCKS).
* **`-Pn`**: Skips ICMP ping discovery (which fails over SOCKS).
* **`-p 22,80,445,3389`**: Scans high-probability service ports to find live hosts quickly without hanging the tunnel.
* **`--open`**: Filters output to display only responsive hosts.

---

### 2. Lightweight Bash Loop via the Tunnel

If Nmap hangs or causes socket exhaustion through the proxy, a simple script using `nc` (netcat) or `curl` can probe a single common port (e.g., port 80 or 22) across the range:

```bash
for i in $(seq 1 254); do
  (nc -z -w 1 --proxy 127.0.0.1:9050 --proxy-type socks5 192.168.98.$i 22 2>/dev/null && echo "192.168.98.$i has port 22 open") &
done
wait

```

---

### 3. Local Execution from the Pivot Host (Most Efficient)

Routing hundreds of connections through an SSH SOCKS proxy introduces high latency. If you have shell access on the intermediary Linux machine, run discovery natively from that host instead:

* **Ping Sweep:**
```bash
for i in $(seq 1 254); do ping -c 1 -W 1 192.168.98.$i >/dev/null && echo "192.168.98.$i is UP"; done

```


* **ARP Cache Inspection:**
```bash
ip neigh
# or
arp -a

```


Checking the ARP table shows systems that have already communicated with the host on that local subnet without generating new probe traffic.
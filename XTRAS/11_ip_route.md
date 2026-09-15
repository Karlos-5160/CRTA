# ip route add 192.168.98.0/24 dev ligolo

This command modifies your Linux kernel's routing table to redirect any network traffic aimed at the `192.168.98.0/24` network into the `ligolo` virtual network interface instead of your standard network card.

### Breakdown of the Components

* **`sudo`**: Executes the command with superuser (root) privileges, which is required to modify kernel network routing tables.
* **`ip`**: The core Linux networking utility used to inspect and configure network devices, IP addresses, and routing tables.
* **`route`**: An object of the `ip` command that specifically manages the kernel's routing table (the internal lookup table Linux uses to decide where to send outgoing network packets).
* **`add`**: The action telling the kernel to insert a brand new entry into that routing table.
* **`192.168.98.0/24`**: The destination network subnet. Any packet whose destination IP begins with `192.168.98.*` (from `.1` to `.254`) matches this rule.
* **`dev`**: Short for **device**. In Linux networking, "device" refers to a network interface (such as a physical Ethernet port like `eth0`, a Wi-Fi card like `wlan0`, or a virtual TUN/TAP adapter). It specifies which network interface should physically (or virtually) transmit the packets matching this route.
* **`ligolo`**: The name of the specific virtual TUN interface you created earlier (`ip tuntap add ... ligolo`).

---

### How It Works Behind the Scenes

Without this command:

* When your Kali machine sends a packet to `192.168.98.10`, the OS looks at its routing table, sees no specific rule for `192.168.98.0/24`, and forwards it to your **default gateway** (your home or lab router via `eth0`). The router drops it because it has no idea how to route into the isolated AD network.

With this command:

* The Linux kernel checks the routing table, sees that `192.168.98.0/24` is bound to the `ligolo` device, and dumps the raw IP packets directly into the `ligolo` virtual interface.
* The Ligolo proxy process intercepts those packets from the interface, encrypts and encapsulates them over your existing TLS connection to the agent on `192.168.80.10`, and the agent transmits them directly onto the internal `192.168.98.0/24` network.
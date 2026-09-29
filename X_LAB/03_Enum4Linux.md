**enum4linux** is a Linux command-line tool used for enumerating information from Windows systems and Samba hosts via SMB (Server Message Block). It is essentially a Perl wrapper around tools like `smbclient`, `net`, `rpcclient`, and `nmblookup`.

It is widely used in network auditing, penetration testing, and Capture The Flag (CTF) challenges to gather reconnaissance data from target machines before deeper assessment.

---

### What It Can Enumerate

* **Users & Groups:** Local accounts, domain accounts, disabled users, and group memberships.
* **Shares:** Listing available share names, permissions (read/write access), and hidden shares.
* **Password Policy:** Minimum password length, lockout thresholds, complexity requirements, and history length.
* **OS & Domain Details:** NetBIOS name, workgroup/domain name, OS version, and whether the host is a Domain Controller.
* **RID Cycling:** Probing Relative Identifiers (RIDs) to uncover hidden usernames or groups when null sessions are restricted.

---

### Common Command Usage

```bash
# Basic run: enumerate all default checks
enum4linux -a <target_IP>

# Enumerate user list only
enum4linux -U <target_IP>

# Enumerate SMB shares
enum4linux -S <target_IP>

# Enumerate password policy information
enum4linux -P <target_IP>

# Perform RID cycling (searches RID range 500-550 to identify users/groups)
enum4linux -r -R 500-550 <target_IP>

# Run with known credentials
enum4linux -u "username" -p "password" -a <target_IP>

```

---

### Key Flags Reference

| Flag | Purpose |
| --- | --- |
| `-a` | Run all simple enumeration checks (default behavior) |
| `-U` | Get user list (`-s` for groups) |
| `-S` | Share enumeration |
| `-P` | Password policy checks |
| `-o` | OS and version detection |
| `-d` | Detailed/verbose output |
| `-r` | RID cycling enumeration |
| `-u` / `-p` | Specify username and password to authenticate |

---

### Modern Alternative: `enum4linux-ng`

The original `enum4linux` was written in Perl and has largely been superseded by **`enum4linux-ng`**, a complete Python 3 rewrite.

* Supports modern protocols and cleaner colored output.
* Adds YAML/JSON structured output export options (`-oJ`, `-oY`).
* Works well with newer Samba/Windows implementations where legacy RPC calls fail.

```bash
# Example using enum4linux-ng
enum4linux-ng -A <target_IP>

```

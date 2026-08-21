# Incident Response Playbook: Systematic SSH Failure Diagnosis (Q83)

This playbook establishes a standard operational procedure (SOP) for troubleshooting remote access failures when a server becomes unreachable via SSH. Rather than guessing the cause, this guide isolates the root issue systematically across four distinct architectural layers.

---

### 🗺️ The Diagnostic Pipeline Architecture

```
[Layer 1: Network] ──> [Layer 2: Port/Firewall] ──> [Layer 3: Service] ──> [Layer 4: Authentication]
```

---

### 🛡️ Layer 1: The Network Layer (ICMP & Routing)
Verify if the underlying virtual machine instance or physical hardware is online, running, and addressable over the network.

* **Diagnostic Action:** Execute a packet internet groper (ICMP) query to check basic routing connectivity.
  ```bash
  ping -c 4 <SERVER_IP>
  ```
* **Analysis:** 
  * *Success:* The host routing configuration is functional. Proceed to Layer 2.
  * *Failure (Timeout / Unreachable):* The instance may be powered off, experiencing a kernel panic, or an edge routing configuration (such as a Cloud Security Group or VPC Access Control List) is actively dropping incoming traffic.

---

### 🛡️ Layer 2: The Port & Firewall Layer (Transport Connectivity)
Verify if the target network port (Default: `22`) is open and accepting traffic through intermediate or host-based firewalls.

* **Diagnostic Action:** Perform a raw TCP handshake audit using `netcat` (`nc`) or `nmap`.
  ```bash
  nc -zv <SERVER_IP> 22
  ```
* **Analysis:**
  * *Success (`Connection to <IP> 22 port [tcp/ssh] succeeded!`):* The transport layer is clear. Proceed to Layer 3.
  * *Failure (`Connection refused` or `Timed out`):* The port is blocked. 
* **Common Resolutions:**
  * Check AWS Security Groups, GCP Firewall Rules, or Azure NSGs to ensure ingress on port 22 is allowed from your source IP.
  * If intermediate firewalls are clear, a host-based firewall on the server might be blocking the port. Once console access is established, audit rules via:
    ```bash
    sudo iptables -L -n -v  # Or: sudo ufw status
    ```

---

### 🛡️ Layer 3: The Service Layer (Daemon Status)
Verify if the OpenSSH daemon (`sshd`) is running and binding to the correct port on the server. *Note: This phase requires Out-Of-Band (OOB) console access via a cloud console dashboard (e.g., AWS EC2 Serial Console), iDRAC, KVM, or an active jump host session.*

* **Diagnostic Action:** Query the supervisor daemon status and examine recent log footprints.
  ```bash
  # 1. Inspect daemon service state
  sudo systemctl status sshd
  
  # 2. Extract service-specific error log streams
  sudo journalctl -u sshd --since "1 hour ago" -p err
  ```
* **Common Resolutions:**
  * If the service is stopped, attempt to initialize it:
    ```bash
    sudo systemctl start sshd
    ```
  * If the service fails to start, verify syntax integrity inside the core daemon configuration file:
    ```bash
    sudo sshd -t
    ```

---

### 🛡️ Layer 4: The Authentication & Authorization Layer
If the network is up, the port is open, and the service is active, the issue lies within keys, permissions, or configuration constraints.

#### **1. Trace the Negotiation Handshake**
Execute the connection command from your local client device using maximum verbosity flags (`-vvv`) to pinpoint the exact moment the handshake drops:
```bash
ssh -vvv <USER>@<SERVER_IP>
```
* *Watch for:* Where the connection halts (e.g., `Permission denied (publickey)`, `Connection closed by remote host`).

#### **2. Inspect Local Permission Matrix Rules**
SSH will fail silently if your private identity key files are overly permissive. Ensure the following client-side settings are enforced:
```bash
chmod 700 ~/.ssh       # Target folder must be private to the owner
chmod 600 ~/.ssh/id_*  # Private keys must read/write only for the owner
```

#### **3. Audit the Target Host Authorization Files**
Once inside via a rescue console, ensure the user's secure directory structure balances matching permissions:
```bash
chmod 700 /home/<USER>/.ssh
chmod 600 /home/<USER>/.ssh/authorized_keys
chown -R <USER>:<USER> /home/<USER>/.ssh
```
*Verify that the public key string inside `~/.ssh/authorized_keys` matches your private key identity exactly.*

#### **4. Review the Global Configuration Daemon Locks**
Inspect `/etc/ssh/sshd_config` for restrictive policy parameters that could block access:

| Directive Parameter | Safe Value Requirement | Impact |
| :--- | :--- | :--- |
| `PasswordAuthentication` | `no` (Production Baseline) | Forces key-only authentication; passwords will fail. |
| `PubkeyAuthentication` | `yes` | Must be explicitly enabled to allow SSH keys. |
| `AllowUsers` | `deploy admin_user` | If configured, *only* listed users can connect. Ensure your username is present. |
| `MaxAuthTries` | `3` | Defines max failed attempts before dropping connection. |

*If any adjustments are committed to `/etc/ssh/sshd_config`, validate syntax structure (`sudo sshd -t`) and reload the service pool safely:*
```bash
sudo systemctl reload sshd
```

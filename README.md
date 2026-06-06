# HTB-Writeups

A structured repository containing detailed writeups, methodology documentation, and system exploitation walkthroughs for various Hack The Box (HTB) machines. 

## 📊 Overview

Each writeup tracks a complete attack lifecycle including reconnaissance, initial access footprinting, internal credential logging, and privilege escalation vectors.

---

## 🖥️ Completed Machines

### 🐧 Linux

| Machine Name | Difficulty | Core Vulnerabilities Exploited | Link |
| :--- | :--- | :--- | :--- |
| **Reactor** | Easy | Next.js RCE (CVE-2025-55182) & Node.js Debug Abuse | [View](./Linux/Easy/Reactor-Easy/writeup.md) |
| **CCTV** | Easy | ZoneMinder Blind SQLi (CVE-2024-51482) & MotionEye Command Execution | [View](./Linux/Easy/CCTV-Easy/writeup.md) |
| **WingData** | Easy | Wing FTP Server RCE (CVE-2025-47812) & Python tarfile.extractall() Privilege Escalation | [View](./Linux/Easy/WingData-Easy/writeup.md) |
| **SmartHire** | Medium | MLflow RCE (CVE-2024-37054) & Python .pth Import Hijacking | [View](./Linux/Medium/SmartHire-Medium/writeup.md) |
| **Helix** | Medium | Apache NiFi RCE (CVE-2023-34468), SSH Key Disclosure & OPC UA Maintenance Mode Privilege Escalation | [View](./Linux/Medium/Helix-Medium/writeup.md) |

### 🪟 Windows

| Machine Name | Difficulty | Core Vulnerabilities Exploited | Link |
| :--- | :--- | :--- | :--- |
| **Bastion** | Easy | SMB Share Information Leakage, VHD Mounting, Offline SAM Extraction & mRemoteNG Decryption | [View](./Windows/Bastion/writeup.md) |

---

_Status: Repository actively maintained and updated upon machine completion._
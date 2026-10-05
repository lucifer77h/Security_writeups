# Nmap Commands Lab: Port Scanning to Privilege Escalation (HackerDNA)

**Platform:** HackerDNA — Learning Lab 102
**Difficulty:** Very Easy
**Category:** Network & Infrastructure
**Flags:** 2/2 (User flag + Root flag)

## Objective
Enumerate a target appliance (no website, services only) using Nmap, identify a weak-credential Telnet service, gain a limited user shell, then escalate to root.

## Recon / Observations
- Target served only a static placeholder page on port 80 — "no website here," other services exist.
- Full port scan with service detection:
  ```
  nmap -sV -p- <target-ip>
  ```
- Found Telnet open. Banner on connect revealed device identity and a direct hint:
  ```
  telnet <target-ip> 23
  ```
  Banner: `MX-200 Console Server — Meridian Systems`, firmware 2.4.1 (legacy branch), with the warning: *"service accounts are still on their factory defaults."*

## Approach
1. The banner's wording ("service accounts") was the actual clue — not a generic `admin/admin` guess. Login as:
   ```
   login: user
   ```
   with a blank/default password worked.
2. Landed in a restricted shell. Prompt told me two things: home directory holds owned files, and admin tasks need the "administrator" account.
3. `ls -la` in home directory showed `flag-user.txt` — read with `cat flag-user.txt` → **User flag captured.**
4. Tried `su administrator` directly — failed: `su: unknown user administrator`. The account name was a guess, not confirmed.
5. Checked actual system accounts:
   ```
   cat /etc/passwd
   ```
   Only `root` and `user` had real login shells (`/bin/sh`) — every other entry was a `nologin` system account. No separate "administrator" account existed; **root was the actual target.**
6. Ran:
   ```
   su root
   ```
   Prompted for password — one of the standard weak-credential guesses (`root`) worked.
7. Now at a `#` prompt (root), checked `/root` directory and `cat`-ed the flag file → **Root flag captured.**

## Vulnerability
- **Weak/default credentials on both a limited service account and the root account** of a network appliance exposed via unauthenticated Telnet (cleartext protocol, no encryption).
- Compounded by cleartext Telnet itself — even if creds were stronger, anything typed in the session (including the root password) crosses the network in plain text, interceptable via sniffing.

## Solution / Fix
- Disable Telnet entirely; use SSH with key-based auth instead.
- Enforce strong, unique passwords on all service and admin accounts — no shared/default creds across appliances.
- Disable direct root login over any remote protocol; require privilege escalation via a named, logged account.
- Audit `/etc/passwd` regularly to confirm no unintended accounts have interactive shells.

## Key Takeaway
Don't guess credentials blind — read the banner/MOTD text carefully first, since labs (and real misconfigured systems) often hand you the actual hint in plain sight ("service accounts... factory defaults"). When an expected account name fails, verify it exists via `/etc/passwd` instead of guessing variations — `root` was the real escalation target the whole time, not a separate "administrator" user.

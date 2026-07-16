# Operating-Security-System
````
# 🛡️ Technical Security Assessment: TryHackMe OS Security
**Date:** 2026-07-03 | **Assessor:** D8701W | **Platform:** TryHackMe
**Target OS:** Linux (Ubuntu) | **Focus:** Credential Hygiene & Privilege Escalation

---

> [!ABSTRACT] Executive Summary
> A localized security assessment was performed on a target Linux host to evaluate user authentication strength and local privilege isolation. The assessment revealed critical vulnerabilities regarding weak user credentials and severe exposure of sensitive passwords within local shell history logs. These systemic flaws allowed an absolute compromise of the host, escalating privileges from a low-level user to `root`.

---

## 🔍 1. Initial Access & System Enumeration

```
### 1.1 Authentication Vector
Initial access to the target environment was established via SSH using the provided low-privileged student credentials.
```bash
# Establishing initial connection as 'sammie'
ssh sammie@10.10.81.141
# Password: dragon
````

### 1.2 User Enumeration & Lateral Movement

During local system reconnaissance, a secondary user account named `johnny` was identified. Based on industry-standard patterns of weak credential reuse, an authentication brute-force attempt was simulated manually using a common top-7 dictionary list.

The account was successfully compromised using the common password string `abc123`.

Bash

```
# Executing lateral movement to the 'johnny' user context
su - johnny
# Password authentication: abc123
```

## 🚀 2. Vulnerability Analysis & Privilege Escalation

### 2.1 Information Disclosure via Shell History Logs

> [!WARNING] VULNERABILITY IDENTIFIED: Plaintext Credential Exposure in Command History
> 
> The `johnny` user account demonstrated poor password hygiene by accidentally executing administrative credentials as commands. Because the shell records standard input strings, administrative secrets were written in plaintext to the user's persistent local history file (`~/.bash_history`).

**Evidence & Enumeration Command:**

Reviewing the environment's command history instantly exposed the system's root administrative password.

Bash

```
# Reviewing execution history log
history
```

_Output Analysis (Line 8):_

Plaintext

```
8  happyHack!NG
```

_(Note: The room context indicated a secondary valid administrative string of `qwertyuiop` was also present via standard task configuration)._

### 2.2 Local Path to Root Compromise

Utilizing the exposed plaintext credentials discovered in the history log, a request to switch the current session context to the absolute system administrator (`root`) was executed.

Bash

```
# Elevating session to root administrative context
su - root
# Password authentication: qwertyuiop

# Verifying host takeover
whoami
# Output confirmed: root
```

### 2.3 Post-Exploitation Proof of Concept

To verify unrestricted read access over sensitive system directories, the root flag was successfully retrieved:

Bash

```
# Retrieving target flag asset
cat /root/flag.txt
```

> [!IMPORTANT] Target Flag Captured
> 
> `THM{YouGotRoot}`

## 🛡️ 3. Remediation & Hardening Recommendations

> [!NOTE] Corrective Actions
> 
> Implementing the following hardening controls will eliminate the attack vectors exploited during this assessment.

1. **Enforce Strong Password Complexities:** Implement PAM (Pluggable Authentication Modules) to enforce robust minimum password lengths and entropy standards, preventing the use of common dictionary strings like `abc123`.
    
2. **Sanitize Shell History Files:** Configure the environment to prevent the logging of accidental entries by adding `HISTCONTROL=ignorespace` to global shell profiles, allowing administrators to prepend a space to commands containing sensitive arguments to avoid log inclusion.
    
3. **Restrict the Sudoers Policy:** Disable direct `su` access to the root account for standard users. Force administrative operations through monitored `sudo` execution configurations to preserve accountability and an immutable audit trail.

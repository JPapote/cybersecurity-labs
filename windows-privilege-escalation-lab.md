# Windows Privilege Escalation Lab

This lab focuses on privilege escalation techniques in Windows systems.

In many penetration tests, attackers initially gain access with limited privileges. The next step is identifying weaknesses that allow them to escalate privileges and gain administrative control over the system.

Privilege escalation in Windows environments often occurs due to misconfigurations, weak permissions, or vulnerable services.

---

# Objective

The goal of this lab is to:

- Understand privilege escalation in Windows environments
- Enumerate system configuration and permissions
- Identify potential escalation vectors
- Gain administrative privileges on the target machine

---

# Tools Used

- Windows command prompt
- PowerShell
- Enumeration commands

---

# Initial Access

After gaining access to the target machine, the first step is identifying the current user and privileges.

```cmd
whoami
whoami /priv
whoami /groups
```
These commands help determine the current user permissions and available privileges.

System Information

Gathering information about the system can reveal potential vulnerabilities.
```
systeminfo
```
Important information includes:

operating system version
installed patches
system architecture

Outdated systems may contain known vulnerabilities.

Running Processes

Checking running processes can reveal services running with high privileges.
```
tasklist
```
This may expose applications that could be abused to escalate privileges.

Installed Services

Enumerating services may reveal misconfigured services or vulnerable software.
```
sc query
```
Look for services running with elevated privileges that might be misconfigured.

Weak Service Permissions

Some Windows services allow modification of their executable path or configuration.

If a user has permission to modify a service binary, it may be possible to replace it with a malicious executable.

Example command to inspect service configuration:
```
sc qc <service-name>
```
Scheduled Tasks

Scheduled tasks sometimes run with high privileges and may execute writable scripts.

Check scheduled tasks:
```
schtasks /query /fo LIST /v
```
Misconfigured tasks may allow privilege escalation.

ritable Files and Directories

Files or directories with weak permissions can sometimes be abused.

PowerShell example:
```
Get-ChildItem "C:\" -Recurse -ErrorAction SilentlyContinue
```
Writable locations may allow attackers to replace files used by privileged processes.

Privilege Escalation Result

After identifying and exploiting a vulnerability, the attacker can gain administrative privileges.

Verify the new privilege level:
```
whoami
```
Expected result:
```
nt authority\system
```
This confirms successful privilege escalation.

Lessons Learned

Privilege escalation vulnerabilities are commonly caused by:

misconfigured services
weak file permissions
vulnerable software
insecure scheduled tasks

Proper system hardening, patch management, and permission control are essential to prevent privilege escalation attacks.

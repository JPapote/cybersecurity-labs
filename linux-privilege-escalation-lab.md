# TryHackMe - Linux Privilege Escalation Lab

This lab focuses on privilege escalation techniques in Linux systems.

In many penetration tests, the attacker initially gains access as a low-privileged user. The next objective is to escalate privileges to obtain root access and fully control the system.

Privilege escalation vulnerabilities are often caused by misconfigurations, insecure permissions, or outdated software.

---

# Objective

The goal of this lab is to:

- Understand the concept of privilege escalation
- Identify misconfigurations in Linux systems
- Enumerate potential privilege escalation vectors
- Gain elevated privileges on the target machine

---

# Tools Used

- Linux terminal
- Basic enumeration commands
- Privilege escalation techniques

---

# Initial Access

After gaining access to the target system, the first step is to identify the current user and system information.

```bash
whoami
id
hostname
```
System Enumeration

System enumeration helps identify potential privilege escalation opportunities.

Check system details:
```
uname -a
cat /etc/os-release
```
Check running processes:
```
ps aux
```
Check network interfaces:
```
ip a
```
Understanding the system configuration helps identify potential weaknesses.

Sudo Permissions

Misconfigured sudo permissions can allow a user to execute commands as root.

Check sudo privileges:
```
sudo -l
```
If a user can run specific commands with sudo privileges, it may be possible to escalate privileges.

SUID Binaries

SUID binaries execute with the permissions of the file owner.

Search for SUID files:
```
find / -perm -4000 2>/dev/null
```
Some SUID binaries can be exploited to gain root privileges if misconfigured.

Writable Files

Files with insecure permissions may allow attackers to modify scripts or binaries.

Check writable files:
```
find / -writable 2>/dev/null
```
Writable configuration files or scripts may be used to escalate privileges.

Cron Jobs

Scheduled tasks can also lead to privilege escalation if misconfigured.

Check cron jobs:
```
cat /etc/crontab
```
If a scheduled task executes a script that is writable by the current user, it may allow command execution as root.

PATH Misconfiguration

If the PATH variable is misconfigured, attackers may place malicious executables in directories that are executed by privileged scripts.

Check PATH:
```
echo $PATH
```
This technique can sometimes be used to execute attacker-controlled commands.

Privilege Escalation Result

After identifying and exploiting a vulnerability, the attacker gains elevated privileges.

Verify root access:
```
whoami
```
Expected result:
```
root
```
This confirms that the privilege escalation was successful.

Lessons Learned

Privilege escalation is one of the most important stages in penetration testing.

Even if the initial access is limited, misconfigurations such as:

insecure sudo permissions
vulnerable SUID binaries
writable scripts
cron job misconfigurations

can allow attackers to gain full control of the system.

Proper system hardening and permission management are essential to prevent these types of attacks.

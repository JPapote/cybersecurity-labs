# TryHackMe - Network Services Lab

This lab focuses on enumerating common network services that are often found during penetration testing.

After identifying open ports with tools such as Nmap, the next step is to interact with the exposed services to gather more information and identify potential vulnerabilities.

Understanding how common services work helps security professionals detect misconfigurations and weak security practices.

---

# Objective

The goal of this lab is to learn how to:

- Identify common network services
- Connect to services running on open ports
- Enumerate information from these services
- Identify potential security weaknesses

---

# Tools Used

- Nmap
- Netcat
- Telnet
- SMB tools
- Linux terminal

---

# Service Enumeration Process

The typical workflow during service enumeration is:

1. Discover open ports
2. Identify the service running on those ports
3. Interact with the service
4. Look for misconfigurations or vulnerabilities

Example Nmap scan:

```bash
nmap -sV <target-ip>
```
This command reveals the service name and version running on open ports.

FTP Enumeration

FTP (File Transfer Protocol) is commonly found on port 21.

Check if anonymous login is allowed:
```
ftp <target-ip>
```
If anonymous login is enabled, it may allow access to files without authentication.

Potential issues:

exposed files
backup files
configuration files

SSH Enumeration

SSH runs on port 22 and is used for secure remote access.

Basic connectivity test:
```
ssh user@<target-ip>
```
While brute force attempts are sometimes tested in labs, the main goal is to identify:

valid usernames
SSH version
potential vulnerabilities
HTTP Enumeration

Web servers usually run on port 80 or 443.

Access the website through a browser or use command line tools.

Example:
```
curl http://<target-ip>
```
Information to look for:

web technologies used

login pages

hidden directories

API endpoints

Directory discovery tools may also be used.

SMB Enumeration

SMB is commonly used for file sharing in Windows environments and usually runs on port 445.

Example command:
```
smbclient -L //<target-ip> -N
```
This may reveal:

shared folders
accessible files
internal resources
Misconfigured SMB shares are a common security issue.

Banner Grabbing

Banner grabbing helps identify service versions and configurations.

Example using Netcat:
```
nc <target-ip> 80
```
Some services display useful information about their version or configuration.

This information may reveal outdated or vulnerable software.

Key Findings

During service enumeration we aim to identify:

available services
accessible resources
software versions
potential entry points

This information helps guide the next phase of a penetration test.

Lessons Learned

Service enumeration is a critical step after discovering open ports.

By interacting with exposed services, attackers and security professionals can identify misconfigurations, weak authentication, and outdated software that may lead to system compromise.

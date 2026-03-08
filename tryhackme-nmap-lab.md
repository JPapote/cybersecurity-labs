# TryHackMe - Nmap Lab

This lab focuses on network reconnaissance using Nmap.  
The goal is to identify open ports, running services, and potential entry points on a target machine.

Network scanning is typically the first step during a penetration test because it provides an overview of the exposed services and possible attack vectors.

---

# Objective

The objective of this lab is to learn how to:

- Discover open ports on a target system
- Identify running services
- Detect service versions
- Perform basic network enumeration

---

# Tools Used

- Nmap
- Linux terminal
- Target machine provided in the lab environment

---

# Initial Scan

The first step is performing a basic scan to discover open ports.

```bash
nmap <target-ip>
```
This scan checks the most common ports and identifies which ones are open.

Example output may reveal services such as:

SSH (22)
HTTP (80)
FTP (21)

This information helps determine potential attack vectors.

Service Version Detection

To identify the versions of services running on open ports, use:
```
nmap -sV <target-ip>
```
This allows us to see more detailed information such as:

web server version
SSH implementation
FTP server software

Knowing the service version can help identify known vulnerabilities.

Aggressive Scan

A more detailed scan can be performed using:
```
nmap -A <target-ip>
```
This enables several features:

OS detection
version detection
script scanning
traceroute

Aggressive scans provide deeper insights into the target system.

Scanning Specific Ports

Sometimes it is useful to scan specific ports.

Example:
```
nmap -p 22,80,443 <target-ip>
```
This limits the scan to selected ports, which can speed up enumeration.

Full Port Scan

To scan all 65535 ports:
```
nmap -p- <target-ip>
```
This may reveal services running on non-standard ports.

Useful Nmap Flags

Commonly used Nmap options include:

-sS : TCP SYN scan
-sV : version detection
-A : aggressive scan
-p- : scan all ports
-T4 : faster scan timing

Example combined scan:
```
nmap -sS -sV -T4 <target-ip>
```
Key Findings

During enumeration we aim to identify:

open ports
running services
service versions
potential vulnerabilities

This information is used to plan the next steps of the penetration test.

Lessons Learned

This lab demonstrates how important network scanning is in the early stages of a security assessment.

Proper enumeration can reveal:

exposed services
misconfigurations
outdated software

These findings often lead directly to exploitation opportunities.

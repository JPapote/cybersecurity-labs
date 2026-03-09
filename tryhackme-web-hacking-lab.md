TryHackMe – Web Hacking Lab
Lab Overview

This lab focuses on identifying and exploiting common web application vulnerabilities based on the OWASP Top 10.

The goal is to understand how attackers exploit web applications and how these vulnerabilities can be detected and mitigated.

Platform: TryHackMe
Category: Web Application Security

Target Enumeration

Before exploiting a web application, it is important to understand the attack surface.

Basic checks include:
```bash
whatweb http://TARGET_IP
```
Directory discovery:
```
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt
```
This helps discover hidden directories and endpoints.

Example findings:
```
/login
/admin
/uploads
```
Testing for SQL Injection

SQL Injection occurs when user input is not properly sanitized.

Example payload:
```
' OR 1=1 --
```
Testing login forms or parameters can reveal authentication bypass vulnerabilities.

Example:
```
username: admin
password: ' OR 1=1 --
```
Result:
Authentication bypass.

Testing for Cross Site Scripting (XSS)
XSS allows attackers to inject malicious scripts into web pages.

Basic test payload:
```
<script>alert("XSS")</script>
```
If the script executes in the browser, the application is vulnerable.

Impact:

Session hijacking
Credential theft
Malicious redirects
File Upload Vulnerabilities

Many applications allow users to upload files.

Improper validation may allow attackers to upload malicious scripts.

Example:
```
shell.php
```
Then access:
```
http://TARGET_IP/uploads/shell.php
```
This can lead to remote code execution.

Post Exploitation

Once access is obtained, attackers may:

retrieve sensitive files
escalate privileges
pivot into the internal network

Example shell connection:
```
nc -lvnp 4444
```
Tools Used

Nmap
Gobuster
Burp Suite
Browser Developer Tools
Netcat

Key Takeaways

Web applications are a common attack vector

Input validation is critical for security

The OWASP Top 10 highlights the most common web vulnerabilities




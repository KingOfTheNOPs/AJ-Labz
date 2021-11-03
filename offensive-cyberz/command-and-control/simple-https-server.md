---
description: Reference for Simple HTTP Server
---

# Simple HTTPS Server

Run script for python Simple HTTPS Server

```
sudo python3 httpsserver.ps
```

{% code title="httpserver.py" %}
```python
from http.server import HTTPServer, SimpleHTTPRequestHandler
import ssl
import socketserver

httpd = socketserver.TCPServer(('PUBLIC IP', 443), SimpleHTTPRequestHandler)

httpd.socket = ssl.wrap_socket(httpd.socket, 
        keyfile="key.pem", 
        certfile='cert.pem', server_side=True)

httpd.serve_forever()
```
{% endcode %}

Create SSL Cert

```bash
openssl req -new -x509 -nodes -out cert.crt -keyout priv.key
cat priv.key cert.crt > cert.pem

```


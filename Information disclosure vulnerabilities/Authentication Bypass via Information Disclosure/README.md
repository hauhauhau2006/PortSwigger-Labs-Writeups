### 1. Vulnerability Summary

Vulnerability: Information Disclosure (via TRACE method) leading to Authentication Bypass.

Description: The web server insecurely enables the HTTP TRACE method, which is designed for diagnostic purposes and echoes back the exact request received by the backend. This reveals hidden HTTP headers appended by the front-end proxy/load balancer. By discovering the specific header used to track the client's IP address, an attacker can spoof this header to impersonate a local user (127.0.0.1) and bypass IP-based access controls.

Impact: Critical. The attacker successfully bypasses the authentication mechanism and gains unauthorized access to the administrative panel, allowing them to perform high-privileged actions (e.g., deleting user accounts).

### 2. Exploitation Steps

*Step 1: Attempt to access the administrative interface by navigating to GET /admin. The server responds with 401 Unauthorized and a message stating: "Admin interface only available if logged in as a local user".*

*Step 2: Send the request to Burp Suite Repeater. Change the HTTP method from GET to TRACE and send the request:*
```
HTTP
TRACE /admin HTTP/2
Host: <target-domain>
```
*Step 3: Analyze the response body. Since TRACE echoes the request, observe that the front-end proxy automatically appended a custom header to track the client's IP. In this lab, it is:
X-Custom-IP-Authorization: <your-public-ip>*

*Step 4: Change the HTTP method back to GET. Manually inject this discovered header into your request and set its value to the localhost IP (127.0.0.1) to trick the backend:*
```
HTTP
GET /admin HTTP/2
Host: <target-domain>
X-Custom-IP-Authorization: 127.0.0.1
```
*Step 5: Send the request. The server accepts the spoofed IP and grants access to the admin panel (200 OK).*

*Step 6: Identify the endpoint to delete the target user (e.g., `/admin/delete?username=carlos`). Send a GET request to this endpoint, ensuring the X-Custom-IP-Authorization: 127.0.0.1 header is still included, to delete the user and solve the lab.*

### 3. Root Cause Analysis (Deep Dive)

Insecure HTTP Methods Enabled: The TRACE method should only be used in development environments. Leaving it enabled on production creates a Cross-Site Tracing (XST) or Information Disclosure risk, as it exposes the internal routing modifications made by reverse proxies.

Flawed Trust in HTTP Headers: The backend application relies on a custom HTTP header (X-Custom-IP-Authorization) for IP-based authentication. HTTP headers are entirely client-controlled. The application blindly trusted this header without verifying the actual TCP/IP layer source address.

### 4. Remediation

Disable the TRACE Method: Configure the web server (Apache, Nginx, IIS) to explicitly block the TRACE and TRACK methods.

Example for Apache: Add TraceEnable Off to the configuration file.

Do Not Rely on Headers for Authentication: Never use HTTP headers (like X-Forwarded-For, X-Real-IP, or custom IP headers) as the sole mechanism for authentication or access control, as they can be easily spoofed.

Secure Reverse Proxy Configuration: If relying on proxy headers is architecturally necessary, the reverse proxy must be configured to strip or overwrite any existing instances of those headers sent by the external client before forwarding the request to the backend.

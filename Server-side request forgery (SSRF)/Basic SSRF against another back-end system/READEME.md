### 1. Vulnerability Summary

**Vulnerability: Server-Side Request Forgery (SSRF) - Internal Network Scanning.**

Description: The application's "Check stock" feature is vulnerable to SSRF via the stockApi parameter. Unlike a local SSRF, the administrative interface is not hosted on localhost but rather on a separate, hidden back-end server within the internal private network (192.168.0.X).

Impact: An attacker can weaponize the application server to perform internal network scanning (Host Discovery), bypass network topology protections, and execute unauthenticated administrative actions (such as deleting users).

### 2. Exploitation Steps

**Step 1: Analyze the Vulnerable Endpoint**

Intercept the `POST /product/stock` request using Burp Suite when clicking the "Check stock" button.

The stockApi parameter takes a URL input, indicating that the backend server makes HTTP requests to fetch stock data.

**Step 2: Internal Network Scanning (Host Discovery)**

Send the intercepted request to Burp Intruder to scan the internal /24 subnet.

Configure the target payload position at the last octet of the IP address:

```
stockApi=[http://192.168.0.](http://192.168.0.)§1§:8080/admin
```

Under the Payloads tab, set the Payload type to Numbers, ranging from 1 to 255 with a step of 1.

Start the attack.

**Step 3: Identify the Back-end Admin Server**

Most requests will fail due to non-existent hosts or closed ports, returning `HTTP 400 Bad Request` or `500 Internal Server Error`.

Sort the Intruder results by the Status or Length column.

Identify the single anomaly—a request that returns an HTTP 200 OK status code. This confirms the exact internal IP address hosting the Admin panel (e.g., 192.168.0.X).

**Step 4: Execute the Unauthorized Action**

Send a new request to Burp Repeater.

Craft the final payload using the discovered internal IP to trigger the user deletion endpoint:
```
stockApi=[http://192.168.0.](http://192.168.0.)[DISCOVERED_IP]:8080/admin/delete?username=carlos
```

The server processes the request and returns an HTTP/2 302 Found redirection, confirming that the user carlos was successfully deleted from the internal system.

### 3. Technical Takeaways

Blind Spots in Perimeter Security: Internal systems often have a weaker security posture because administrators assume they are protected by the outer firewall. This lab demonstrates that SSRF effectively turns the public-facing web server into a Trojan horse, entirely bypassing perimeter defenses.

Error-Based Scanning: In penetration testing, HTTP errors (400/500) are not just roadblocks; they are valuable signals. Distinguishing between a connection timeout (dead IP) and a successful connection (live IP) allows attackers to map out the entire internal network topology blindly.

### 4. Remediation

Network Segmentation: Implement strict network access control lists (ACLs) to ensure the web-facing application server can only communicate with required back-end systems on specific ports, completely blocking access to administrative subnets.

Strict Input Validation: Enforce a hardcoded allow-list (whitelist) of approved URLs or use internal mapping IDs instead of passing full, user-controllable URLs in HTTP requests.

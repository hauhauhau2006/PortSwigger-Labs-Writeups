### 1. Vulnerability Summary

Vulnerability: Server-Side Request Forgery (SSRF) bypassing strict whitelist filters via URL Parsing Inconsistencies.

Description: The application's stockApi parameter attempts to prevent SSRF by implementing a strict whitelist. The filter mandates that the provided URL must contain the authorized domain `(stock.weliketoshop.net)`. However, the front-end security filter and the back-end HTTP client parse URLs differently. By manipulating the URL structure using credentials `(@)` and double-encoded URL fragments `(#)`, an attacker can satisfy the whitelist filter while forcing the backend to connect to a different, internal host.

Impact: Bypass of perimeter security and input validation, leading to unauthorized access to the internal localhost administrative interface and the ability to execute privileged actions.

### 2. Exploitation Steps

**Step 1: Identifying the Whitelist**

Intercept the POST /product/stock request using Burp Suite.

Modifying the stockApi parameter to a generic internal IP `(http://127.0.0.1/)` results in an `HTTP 400 Bad Request`.

Observing the default request reveals the required whitelisted domain: `stock.weliketoshop.net`. The filter checks if this exact string exists within the supplied URL.

**Step 2: URL Structure Manipulation**

To bypass the filter, the required domain must be included, but the actual connection must be redirected. We leverage the URL credentials syntax: `http://username@hostname/`.

Sending `stockApi=http://username@stock.weliketoshop.net/` returns an `HTTP 500 Internal Server Error`, indicating the request bypassed the 400-level filter and reached the backend.

**Step 3: Appending the Fragment Identifier**

To force the backend to ignore the whitelisted domain, we introduce the fragment identifier `(#)`. Everything after the `#` is typically ignored by HTTP clients when establishing a connection.

Sending `http://localhost#@stock.weliketoshop.net/` is blocked `(400 Bad Request)` because the front-end filter recognizes the structural anomaly.

**Step 4: The Double Encoding Bypass**

We apply Double URL Encoding to the # character to sneak it past the front-end filter.

`#` -> %23 (First encode)

`%`-> %25 (Second encode) -> Final: %2523

We construct the final payload to access the admin panel and delete the target user:
```
stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```
The server responds with an `HTTP/2 302 Found`, successfully deleting the user.

### 3. Technical Takeaways: The Parser Differential

This vulnerability is a textbook example of a Parser Differential (or Parser Inconsistency). The exploit succeeds because two different components in the architecture process the same string differently:

The Front-end Security Filter's View:

It decodes the URL once, seeing: `http://localhost%23@stock.weliketoshop.net/`

It interprets `@` as the delimiter for credentials. Thus, it assumes the hostname is `stock.weliketoshop.net` and the username is `localhost%23`.

Conclusion: The hostname matches the whitelist. The request is authorized.

The Back-end HTTP Client's View (The Execution):

The backend receives the authorized string and decodes it a second time, resolving the URL to: `http://localhost#@stock.weliketoshop.net/`

When establishing the TCP/IP connection, the HTTP client encounters the # symbol. According to URL specifications (RFC 3986), the # denotes a fragment identifier.

The client truncates the URL at the `#`, completely ignoring the `@stock.weliketoshop.net/ portion.`

Conclusion: The backend establishes a connection directly to `http://localhost`, executing the SSRF payload.

### 4. Remediation
Avoid Passing URLs as Input: The most robust defense is to avoid taking full URLs from the user. Instead of `stockApi=http://...,` use an identifier (e.g., productId=1) and map it to internal URLs strictly on the server side.

Parser Synchronization: Ensure that the URL parsing library used by the security filter behaves exactly the same as the HTTP client library used by the backend. They must decode and evaluate URL structures identically.

Defense in Depth: Internal systems (like Admin panels) must not rely solely on network location (e.g., "trusting localhost") for authentication. Robust access controls should be enforced at the application layer regardless of the request's origin.

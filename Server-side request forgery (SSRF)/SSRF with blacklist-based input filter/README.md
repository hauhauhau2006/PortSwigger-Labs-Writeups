### 1. Vulnerability Summary

Vulnerability: Server-Side Request Forgery (SSRF) bypassing weak blacklist filters.

Description: The application is vulnerable to SSRF via the stockApi parameter. The developers attempted to patch this by implementing a blacklist defense that blocks common local hostnames (e.g., 127.0.0.1, localhost) and sensitive endpoints (e.g., /admin). However, due to inconsistencies in how different parsers handle URL encoding and IP representations, these filters can be trivially bypassed.

Impact: An attacker can circumvent the security filters to access the internal administrative panel and execute unauthenticated, privileged actions (such as deleting a user).

### 2. Exploitation Steps

**Step 1: Discovering the Filter**

Intercept the `POST /product/stock` request using Burp Suite.

Attempting a basic local SSRF payload `(stockApi=http://127.0.0.1/)` results in the request being blocked, indicating a blacklist filter is in place.

**Step 2: Bypassing the IP Blacklist**

To evade the IP restriction, modify the payload to use an alternative IP representation: `stockApi=http://127.1/.`

The server accepts this input, demonstrating that the filter only performs basic string matching and fails to account for network-layer IP normalization.

**Step 3: Bypassing the Path Blacklist**

Attempting to access the admin panel `(stockApi=http://127.1/admin)` triggers the filter again.

To bypass the `/admin` string check, apply Double URL Encoding to the character `a`.

The letter `a` becomes `%61`.

Encoding the `%` symbol again results in `%25`.

The final encoded payload is `%2561.`

Send the payload: `stockApi=http://127.1/%2561dmin.` The server returns a `200 OK status` along with the HTML of the admin panel.

**Step 4: Executing the Unauthorized Action**

Locate the endpoint for deleting the user carlos within the HTML response: `/admin/delete?username=carlos.`

Because HTTP is stateless and the filter evaluates every single request independently, the blacklist evasion must be maintained.

Send the final payload with the double-encoded path to execute the deletion:
```
stockApi=http://127.1/%2561dmin/delete?username=carlos
```

The server returns an HTTP/2 302 Found redirection, confirming the successful deletion of the user.

### 3. Technical Takeaways (Deep Dive)

This lab perfectly illustrates why blacklist-based input validation is fundamentally flawed:

Network-Layer Zero Compression `(127.1)`: The blacklist looks for the exact string `127.0.0.1`. However, networking stacks (like the OS networking API) are designed to be flexible and will automatically expand omitted octets with zeros. 127.1 resolves to 127.0.0.1 at the network layer, successfully bypassing the application-layer string filter.

Parser Inconsistencies & Double URL Encoding `(%2561dmin)`: * The front-end security filter performs a single URL decode on the input, seeing `%61dmin`. Since `%61dmin` does not match the blacklisted string admin, the filter allows the request to pass.

The back-end HTTP client, however, performs a second URL decode when initiating the actual internal request, converting `%61dmin` back to admin.

Note: This bypass is not limited to the character a. Encoding any character in the blacklisted word (e.g., encoding `d` to `%2564` resulting in `a%2564min`) would be equally effective in breaking the string signature.

Statelessness of HTTP and Security Filters: Successfully viewing the admin panel does not establish an authenticated `"session"` that exempts the attacker from future checks. The blacklist filter sits at the API gateway and scrutinizes every incoming stockApi parameter. Therefore, the double-encoding evasion must be reapplied to the final deletion request to pass through the gateway a second time.

### 4. Remediation

Adopt Allow-listing (Whitelisting): Instead of trying to guess and block all possible bad inputs (which is mathematically impossible due to encoding variations), define a strict list of allowed internal hostnames or URLs.

Consistent Parsing: Ensure that the URL parser used by the security filter behaves exactly the same as the HTTP client that makes the back-end request.

Defense in Depth: Do not rely solely on perimeter filters. The internal Admin panel should still require strong authentication and authorization, regardless of whether the request originates from localhost.

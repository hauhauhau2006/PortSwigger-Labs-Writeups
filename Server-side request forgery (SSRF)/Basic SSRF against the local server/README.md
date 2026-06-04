### 1. Vulnerability Summary

**Vulnerability: Server-Side Request Forgery (SSRF)**

*Description: The application's "Check stock" feature accepts a full URL as input via the stockApi parameter. Due to a lack of input validation, an attacker can manipulate this parameter to force the backend server into making arbitrary HTTP(S) requests to internal systems that are normally inaccessible from the external network.*

*Impact: This vulnerability allows an attacker to bypass Access Control mechanisms, gaining unauthorized access to the internal Admin panel and executing privileged actions.*

### 2. Exploitation Steps

**Step 1: Analyze the "Check stock" feature:**

Access any product page and use the Check stock functionality.

Intercept the `POST /product/stock` request using Burp Suite.

Observe that the application sends a stockApi parameter containing a full URL (e.g., `http://stock.weliketoshop.net:8080/product/stock/check?...`). This indicates the backend server is fetching data directly from the user-supplied URL.

**Step 2: Access the Admin Panel via Localhost**

To test for SSRF, modify the value of the stockApi parameter to point to the server's loopback address: `http://localhost/admin`.

Send the request and observe the response. The server returns the HTML source code of the internal Admin panel, proving that the application blindly trusts and executes local requests.

**Step 3: Identify the endpoint to delete a user**

Review the HTML response obtained in Step 2.

Locate the hyperlink (href attribute) corresponding to the action for deleting the user carlos, which is: `/admin/delete?username=carlos`.

**Step 4: Execute the unauthorized action**

Send the request to Burp Repeater for easier manipulation.

Modify the stockApi parameter to construct the full URL pointing to the deletion endpoint on the local server:
```
stockApi=http://localhost/admin/delete?username=carlos
```
Send the request. The system returns an `HTTP/2 302 Found` status code redirecting back to /admin (as shown in the provided screenshot). This confirms that the user carlos was successfully deleted by leveraging the server's internal privileges.

### 3. Technical Takeaways
   
This vulnerability stems from two primary architectural flaws:

Missing Input Validation: The server blindly trusts URL input from the client-side without implementing an allow-list (whitelist) to restrict permitted domains or endpoints.

Perimeter-based Security Flaw: The internal administrative system relies solely on perimeter defense, trusting any request originating from localhost or internal IPs while skipping user authentication. SSRF allows the attacker to use the backend server as a proxy to bypass this firewall.

### 4. Remediation

Implement an Allow-list (Whitelist): Ensure the stockApi parameter only accepts specific, pre-approved identifiers or hostnames rather than arbitrary, fully qualified URLs.

Disable HTTP Redirections: Configure the backend HTTP client to drop or strictly validate unexpected redirects to prevent SSRF escalation.

Adopt Zero Trust Architecture: Enforce strict authentication and authorization checks for sensitive endpoints (like the Admin panel), regardless of whether the request originates from the external network or localhost.

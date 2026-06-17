### 1. Vulnerability Summary

Vulnerability: Information Disclosure (Timing Attack) leading to Username Enumeration & Rate Limit Bypass.

Description: The application's login mechanism is vulnerable to a timing-based side-channel attack. It only hashes the provided password if the submitted username actually exists in the database. By sending an excessively long password, an attacker can amplify the hashing time, creating a measurable delay that reveals valid usernames. Furthermore, the application relies on the client-controllable X-Forwarded-For HTTP header for its IP-based rate limiting, allowing attackers to completely bypass brute-force protections.

Impact: High. The combination of these vulnerabilities allows an attacker to reliably enumerate valid usernames and subsequently brute-force their passwords without being blocked, ultimately leading to Account Takeover (ATO).

### 2. Exploitation Steps

**Phase 1: Username Enumeration (Timing Attack)**

*Step 1: Intercept a failed login request (POST /login) using Burp Suite.*

*Step 2: Change the protocol from `HTTP/2` to `HTTP/1.1` to prevent request multiplexing, ensuring accurate timing measurements.*

*Step 3: Inject the X-Forwarded-For header to spoof the IP address and bypass rate limiting. Modify the password to an extremely long string (e.g., 100+ characters) to amplify the server's processing time.*
```
HTTP
POST /login HTTP/1.1
Host: <target-domain>
X-Forwarded-For: §1§
...
username=§candidate_name§&password=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...
```
*Step 4: Send the request to Burp Intruder. Select the Pitchfork attack type to iterate both payloads simultaneously.*

*Step 5: Configure the payloads:*

Payload 1 (IP Spoofing): Numbers type (from 1 to 500, step 1).

Payload 2 (Usernames): Simple list using the provided candidate usernames.

*Step 6: Under Intruder Columns settings, enable Response received to monitor the server's delay.*

*Step 7: Start the attack. Sort the results by the Response received column. The request with a significantly higher delay (e.g., ~1200ms compared to ~50ms for others) reveals the valid username (e.g., carlos).*

**Phase 2: Password Brute-forcing**

*Step 1: Return to the Intruder Positions tab. Hardcode the discovered valid username and move the second payload marker to the password field:*
```
HTTP
X-Forwarded-For: §1§
...
username=carlos&password=§candidate_password§
```
*Step 2: Keep the attack type as Pitchfork to ensure a fresh spoofed IP is used for every password attempt.*

*Step 3: Update Payload 2 with the provided password dictionary.*

*Step 4: Start the attack. Monitor the Status column. The request that returns a `302 Found` (redirecting to /my-account) instead of `200 OK` indicates the correct password.*

*Step 5: Log in with the discovered credentials to complete the lab.*

### 3. Root Cause Analysis

Asymmetric Processing Time: The backend logic verifies the existence of the username before initiating the computationally expensive password hashing process (e.g., bcrypt/Argon2). If the username does not exist, the server returns an error immediately, creating a distinct timing difference.

Insecure Rate Limiting: The application implements its account lockout/rate-limiting policy based on the `X-Forwarded-For` HTTP header instead of the actual TCP/IP socket connection. Since HTTP headers are fully controllable by the client, the protection is completely ineffective.

### 4. Remediation

Uniform Processing Time: The login function must execute in a constant amount of time regardless of whether the username is valid or invalid. If a user does not exist, the backend should compute a dummy hash of the same complexity to mask the timing difference.

Robust Rate Limiting: Never rely on easily spoofable HTTP headers (like X-Forwarded-For or X-Real-IP) as the sole metric for security controls. Implement robust rate limiting based on the true network-layer IP address, and enforce additional protections such as CAPTCHAs, device fingerprinting, or Multi-Factor Authentication (MFA) after consecutive failed attempts.

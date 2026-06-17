### 1. Vulnerability Summary

Vulnerability: Broken Logic in Rate Limiting (Brute-force Protection Bypass).

Description: The application implements an IP-based blocking mechanism that locks out an IP address after three consecutive failed login attempts. However, the underlying logic is fundamentally flawed: any successful authentication event from that IP address resets the failed attempt counter back to zero. An attacker possessing one set of valid credentials can exploit this by interleaving successful logins with brute-force attempts against a target account, effectively bypassing the IP block entirely.

Impact: High. The broken protection mechanism fails to prevent credential stuffing and brute-force attacks, leading to the compromise of targeted user accounts (Account Takeover).

### 2. Exploitation Steps

*Step 1: Verify the protection mechanism. Attempt to log in with incorrect credentials three times. Observe that the IP is temporarily blocked. Wait for the block to expire.*

*Step 2: Log in successfully using the provided valid credentials `(wiener:peter)`. Observe the `302 Found response`.*

*Step 3: Open Burp Suite and navigate to Settings > Sessions > Macros. Click Add and select the successful `POST /login` request for wiener from the HTTP history. Name the macro Reset IP.*

*Step 4: Navigate to Settings > Sessions > Session handling rules. Click Add to create a new rule.*

*Step 5: In the Rule Actions tab, click Add > Run a macro and select the Reset IP macro.*

*Step 6: In the Scope tab, check the Intruder tool and select Include all URLs under the URL scope.*

*Step 7: Send a login request for the target user (carlos) with an arbitrary password to Burp Intruder.*

*Step 8: Set the attack type to Sniper and place the payload marker entirely on the password value:*
```
username=carlos&password=§guess§
```
*Step 9: In the Payloads tab, paste the provided list of candidate passwords.*

*Step 10: Crucial Step: Navigate to the Resource Pool tab, create a new resource pool, and set the Maximum concurrent requests to 1. This ensures the requests are sent sequentially, allowing the macro to properly reset the counter before each brute-force attempt.*

*Step 11: Start the attack. Sort the results by the Status column. The request returning a `302 Found` indicates the correct password (sunshine).*

*Step 12: Log in via the web interface using carlos:sunshine to access the target account and solve the lab.*

### 3. Root Cause Analysis

Flawed State Management: The application's security state (the failed attempt counter) is loosely tied to the IP address and is explicitly cleared upon any successful authentication from that IP. The developers failed to isolate the state track per user account or enforce an absolute, non-resettable time penalty for consecutive failures.

### 4. Remediation
Implement Strict Time-Based Lockouts: Once an IP address triggers the lockout threshold (e.g., 5 failed attempts in 5 minutes), apply an absolute block for a specific duration (e.g., 15 minutes). A successful login to a different account must never reset this timer.

Dual-Layer Rate Limiting: Implement rate limiting on both the IP address level and the Username level. If an attacker routes traffic through thousands of IPs (bypassing the IP block), the per-user block will still protect the targeted account.

Use Standard Anti-Automation Defenses: Supplement rate limiting with adaptive CAPTCHAs (like reCAPTCHA v3) that trigger after a few failed attempts, or implement multi-factor authentication (MFA) to render password-guessing attacks ineffective.

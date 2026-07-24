# Lab Write-up: OS Command Injection, Simple Case

---

## 1. Lab Overview
* **Platform:** PortSwigger Web Security Academy
* **Vulnerability Type:** OS Command Injection
* **Difficulty Level:** Apprentice
* **Objective:** Exploit the command injection vulnerability to execute the `whoami` command, determine the current user's name, and solve the lab.

---

## 2. Tools Required
* A modern Web Browser
* Burp Suite (Community or Professional Edition)

---

## 3. Exploitation Steps

### Step 1: Intercept the Target Request
* Navigate to the lab environment and click on "View details" for any product.
* Ensure Burp Suite is running and intercepting HTTP traffic.
* Click the "Check stock" button located at the bottom of the page.
* Locate the `POST /product/stock` request in Burp Suite and send it to Repeater (Ctrl + R).

### Step 2: Analyze the Request
The intercepted request sends user input via an HTML form. The body looks like this:

```http
POST /product/stock HTTP/2
Host: 0a7b006c03d929f080434e8d005b00c5.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 21

productId=2&storeId=2
```

Step 3: Inject the Payload
The backend application takes the productId and storeId parameters and passes them directly to a shell command (e.g., stock_check.sh 2 2).

To exploit this, we use the pipe metacharacter (|) to chain our own command. Modify the storeId parameter in Burp Repeater to include |whoami:
```
productId=2&storeId=2|whoami
```

Step 4: Execute and Observe
Send the modified request. The application will execute the stock check, pipe the output, and then execute our injected whoami command.

The HTTP response will return the username of the system process (e.g., peter-X5v) instead of the standard stock count:
```

HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 9

peter-X5v
```
4. Remediation & Conclusion

This vulnerability demonstrates the severe risks of passing unsanitized user input into system shells. To mitigate OS Command Injection:

Avoid System Calls: Never call OS commands directly from the application layer if built-in language APIs or libraries can achieve the same result.

Input Validation: If system calls are absolutely necessary, implement strict allow-listing for all user input.

Parameterization: Use safe APIs that allow command execution with explicit parameterization, preventing shell interpretation of metacharacters.



<img width="959" height="452" alt="Screenshot 2026-07-24 135220" src="https://github.com/user-attachments/assets/5e4dbbc2-3b80-4965-a139-5ccd50ca6fbd" />

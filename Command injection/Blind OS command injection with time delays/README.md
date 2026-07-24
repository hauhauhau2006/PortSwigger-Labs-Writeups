# Lab Write-up: Blind OS Command Injection with Time Delays

---

## 1. Lab Overview
* **Platform:** PortSwigger Web Security Academy
* **Vulnerability Type:** Blind OS Command Injection (Time-based)
* **Difficulty Level:** Practitioner
* **Objective:** Exploit a blind OS command injection vulnerability in the feedback feature to cause a 10-second time delay.

---

## 2. Tools Required
* A modern Web Browser
* Burp Suite (Community or Professional Edition)

---

## 3. Vulnerability Analysis
Unlike the "Simple Case" where command output is reflected in the HTTP response, this application does not return the output of the executed command. However, the application handles user input in a way that executes system commands asynchronously or silently. 

To verify the vulnerability, we must rely on **time-based inference**. By injecting a command that pauses execution (e.g., `ping`), we can measure the time it takes for the server to respond. If the response is delayed by the exact amount of time we specified, we can confirm the injection is successful.

---

## 4. Exploitation Steps

### Step 1: Intercept the Target Request
* Navigate to the lab environment and click on the "Submit feedback" page.
* Fill out the form with dummy data (Name, Email, Subject, Message) and click submit.
* In Burp Suite, go to the **Proxy > HTTP history** tab and locate the `POST /feedback/submit` request.
* Send this request to **Repeater** (Ctrl + R).

### Step 2: Analyze the Request
The body of the `POST /feedback/submit` request uses the `application/x-www-form-urlencoded` format. It looks like this:

```http
POST /feedback/submit HTTP/2
Host: <YOUR-LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=YOUR_CSRF_TOKEN&name=test&email=test%40gmail.com&subject=test&message=test
```

Step 3: Craft and Inject the Payload
Through fuzzing the parameters, we discover that the email parameter is vulnerable. We will use the || (OR) logical operator to chain our command, ensuring it executes even if the preceding command fails.

The payload we want to execute is:
|| ping -c 10 127.0.0.1 ||

Because the request is x-www-form-urlencoded, we must URL-encode the payload to prevent the web server from misinterpreting the spaces and special characters. The URL-encoded payload becomes:
%7C%7Cping+-c+10+127.0.0.1%7C%7C

Modify the email parameter in Burp Repeater:

```
csrf=YOUR_CSRF_TOKEN&name=test&email=test%40gmail.com%7C%7Cping+-c+10+127.0.0.1%7C%7C&subject=test&message=test
```

Step 4: Execute and Observe
Click Send in Burp Repeater.

Observe the response timer in the bottom right corner of Burp Suite.

The application will take approximately 10 seconds (10,000+ milliseconds) to return a 200 OK response, confirming that the ping command was successfully executed by the backend server.

Return to the browser, and the lab will be marked as Solved.

5. Remediation
To prevent Blind OS Command Injection:

Avoid System Calls: Do not use OS commands to process user input. Use native library APIs (e.g., built-in mail libraries instead of OS-level mail commands).

Strict Input Validation: If passing input to a system command is unavoidable, validate the input strictly against a narrow allow-list (e.g., ensuring an email address only contains valid alphanumeric and specific symbol characters, rejecting all shell metacharacters).

Parameterization: Use safe, parameterized execution methods where the OS command and its arguments are passed separately, preventing the shell from evaluating user input as executable code.
<img width="956" height="436" alt="Screenshot 2026-07-24 143732" src="https://github.com/user-attachments/assets/54e4eedb-2abe-4d38-8f54-2b4520d1238e" />

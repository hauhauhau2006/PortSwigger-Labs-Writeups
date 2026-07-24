1. Overview
Vulnerability Type: Blind OS Command Injection.

Objective: Execute the whoami command on the server and retrieve the output.

Vulnerable Point: The "Submit Feedback" feature, specifically the email parameter, which is passed directly to the operating system shell without proper sanitization.

2. Vulnerability Analysis
When a user submits feedback, the application returns an HTTP 200 OK response regardless of whether the injected bash command executes successfully or not. Because the execution happens asynchronously or silently (Blind), we cannot read the command output directly from the HTTP response.

To extract the data, we must use the output redirection operator (>) to force the server to write the command's result into a public directory. In this specific application, the directory storing product images (/var/www/images/) is an ideal target because its contents can be accessed directly via the browser.

3. Exploitation Steps
Step 1: Intercept the Feedback Submission Request
Use an interception proxy (such as Burp Suite) to capture the HTTP POST request generated when submitting a feedback form. In the request body, focus on the email parameter.

Step 2: Prepare the Payload
We need a payload that accomplishes three things:

Terminates the server's original command (using || or ;).

Executes our desired command (whoami).

Redirects the output to the image directory (> /var/www/images/output.txt).

The raw payload looks like this:
|| whoami > /var/www/images/output.txt ||

Since the payload is sent via an HTTP POST request (application/x-www-form-urlencoded), all special characters (spaces, |, >) must be URL-encoded so the backend parses them correctly:

| becomes %7c

Space becomes + or %20

> becomes %3e

URL-Encoded Payload:
```
HTTP
email=test%40example.com%7c%7cwhoami+%3e+/var/www/images/output.txt%7c%7c

```
Step 3: Execute the Payload
Send the modified request to the server. The server will respond with a 200 OK ("Thank you for submitting feedback"). At this point, the output.txt file has been silently created on the backend.

Step 4: Data Extraction (Reading the File)
Return to the application's home page, right-click on any product image, and select "Open image in new tab".
The original image URL will look like this:
`https://<YOUR-LAB-ID>.web-security-academy.net/image?filename=1.jpg`

Replace the image filename with the text file we just created:
`https://<YOUR-LAB-ID>.web-security-academy.net/image?filename=output.txt`

The browser will display the output of the command:
www-data (or the corresponding system user). The lab is now solved!

🛡️ Remediation & Defensive Best Practices
To prevent OS Command Injection vulnerabilities in production environments, developers must implement the following defenses:

Avoid Direct OS Command Calls: Minimize or eliminate the use of shell execution functions (e.g., system(), exec(), shell_exec() in PHP, or os.system() in Python). Instead, use built-in language APIs or dedicated libraries (e.g., using a native Mailer library instead of invoking the OS mail command).

Strict Input Validation: Implement robust allow-listing or regex validation for all user input. A valid email parameter must never contain shell metacharacters such as |, ;, &, or $.

Use Parameterized Execution: If invoking OS commands is absolutely necessary, pass user input as safely escaped arguments rather than concatenating strings directly into the command line execution context.
<img width="952" height="455" alt="image" src="https://github.com/user-attachments/assets/c5f2f1af-306f-408d-ba79-e9b2871cbd17" />
<img width="956" height="462" alt="Screenshot 2026-07-24 151206" src="https://github.com/user-attachments/assets/fa7799af-82c8-46fa-9ea0-36a654aed358" />


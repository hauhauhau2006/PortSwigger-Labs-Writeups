### 1. Vulnerability Summary

Vulnerability: Information Disclosure via Exposed Debug Endpoint.

Description: A forgotten HTML comment on the application's homepage reveals a hidden path to a PHP debugging script (phpinfo.php).

Impact: High. The phpinfo() function outputs extensive information about the current state of PHP, including OS version, server configuration, loaded extensions, and most critically, Environment Variables. Exposing variables like SECRET_KEY allows attackers to forge authentication tokens (e.g., JWT) or escalate privileges.

### 2. Exploitation Steps

*Step 1: Navigate to the homepage of the target application.*

*Step 2: View the page source (Ctrl + U) or inspect the HTTP response using Burp Suite.*

*Step 3: Search the HTML source code for keywords like debug, test, or simply scroll to the bottom.*

*Step 4: Discover the leftover HTML comment:*
``

*Step 5: Access the exposed endpoint by appending the path to the base URL:*
`GET /cgi-bin/phpinfo.php HTTP/2`

*Step 6: The server renders the standard phpinfo() page.*

*Step 7: Search (Ctrl + F) for the SECRET_KEY within the "Environment" section and extract its value to complete the lab.*

### 3. Root Cause Analysis

Leftover Debugging Code: Developers often create utility scripts or comments during the development phase. Failure to strip these artifacts before deployment causes sensitive endpoints to be exposed to the public.

Unrestricted Access: Sensitive endpoints (like phpinfo.php) were left accessible without any authentication mechanism or IP whitelisting.

### 4. Remediation

Remove Debug Scripts: Never deploy files like phpinfo.php, test.php, or debug consoles to the production server. Delete them completely.

Code Minification & Stripping: Integrate tools into the CI/CD pipeline to automatically strip HTML comments and development logs before the code is built and deployed.

Access Control: Restrict access to administrative or monitoring directories using strong authentication and IP whitelisting.

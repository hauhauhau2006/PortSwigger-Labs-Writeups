### Lab: Web shell upload via Content-Type restriction bypass (PortSwigger Web Security Academy)

### Objective: Upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret.

### The Concept

This vulnerability occurs when a web server relies strictly on the Content-Type header provided in the HTTP request to validate file uploads, rather than verifying the actual contents or the file extension.

Instead of taking the long route (uploading a valid image and then rewriting the entire filename and body), we can take a much faster, more direct approach: upload the malicious PHP file directly and simply forge its "ID card" (the Content-Type header).

**Step-by-Step Exploitation**

*Step 1: Prepare the Payload*

First, create a malicious PHP file on your local machine named shell.php. This file contains a simple script to read the target secret:
```
PHP
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
*Step 2: Intercept the Upload*

Log into the application using the provided credentials (wiener:peter).

Navigate to the My Account page to access the avatar upload functionality.

Turn on Intercept in Burp Suite Proxy.

Select your local shell.php file and click Upload.

*Step 3: The Bypass (Forging the Content-Type)*

Burp Suite will catch the POST /my-account/avatar request. If you look at the raw request, it will look something like this:
```
HTTP
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
If we send this as-is, the server will block it because application/x-php is not an allowed image type.
To bypass the filter, simply modify the Content-Type header to pretend it's a PNG image:

Change:
```
Content-Type: application/x-php
To:
Content-Type: image/png
```

Leave the filename="shell.php" and the payload exactly as they are. Click Forward (or Send if you are in Repeater).

The server checks the Content-Type, sees image/png, and blindly trusts it, saving our file as shell.php.

*Step 4: Execute the Web Shell*

Now that the payload is successfully planted on the server, we need to trigger it.

Go back to your browser and turn off Burp Intercept.

Reload the My Account page. You will likely see a broken image icon for your avatar.

Right-click the broken avatar image and select "Open image in new tab" (or use "Inspect Element" to find the image source).

The URL will look something like: https://YOUR-LAB-ID.web-security-academy.net/files/avatars/shell.php

Because the file retains its .php extension, the backend web server (e.g., Apache) will execute our PHP code rather than serving it as an image. The page will load and display Carlos's secret string. Submit this string to solve the lab!

💡 Key Takeaway
Validation must happen on the server side using robust methods (like checking file signatures/magic bytes and restricting execution directories). Never trust user-controlled inputs, especially HTTP headers like Content-Type, as they can be trivially manipulated by attackers.

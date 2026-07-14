

**Lab:** Web shell upload via obfuscated file extension (PortSwigger)

**Objective:** Bypass the file upload filter using a null byte, upload a PHP web shell, and exfiltrate the contents of `/home/carlos/secret`.

**Methodology:** File Extension Obfuscation (Null Byte `%00` Injection) + Web Shell

## The Vulnerability

The web application attempts to validate file uploads by checking if the filename ends with an allowed extension, such as `.jpg` or `.png`.

However, the backend system (often using C-based APIs for file handling) treats the URL-encoded null byte character (`%00`) as a string terminator.

When an attacker uploads a file named `shell.php%00.jpg`, the web application's high-level validation sees the `.jpg` extension and allows the upload. 

But when the filename is passed to the underlying operating system to be saved, the string is truncated at the null byte. 

As a result, the `.jpg` extension is discarded, and the file is saved to the disk purely as `shell.php`, allowing the attacker to execute arbitrary PHP code.

## Exploitation Steps

### Step 1: Intercept the Upload Request

1. Log in to the application using the provided credentials (`wiener:peter`).

2. Navigate to the **My Account** page.

3. Turn on **Intercept** in Burp Suite Proxy.

4. Select a malicious PHP file containing the following web shell payload and click **Upload**:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Step 2: Inject the Null Byte
Once Burp Suite intercepts the POST /my-account/avatar request, locate the Content-Disposition header in the request body.

Modify the filename parameter by appending the null byte %00 followed by a valid image extension like .jpg.

The modified payload section should look like this:
```
Content-Disposition: form-data; name="avatar"; filename="shell.php%00.jpg"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Send the request. The server responds with 200 OK and a confirmation message in the response body: The file avatars/shell.php has been uploaded.

This response confirms that the filter was bypassed and the operating system successfully truncated the filename at the null byte.

Step 3: Execute the Payload
Because the file was saved on the server without the .jpg extension, we must request the truncated filename to execute it.

Open a browser or use a GET request in Burp Suite to navigate directly to the uploaded PHP file:
```
https://[YOUR-LAB-ID].web-security-academy.net/files/avatars/shell.php
```
The backend Apache server recognizes the .php extension and executes the script.

The page will load and display Carlos's random secret string. Copy this string and submit it to solve the lab.

Mitigation
To prevent null byte injection and file extension obfuscation, developers should:

Sanitize all user-supplied filenames by stripping null bytes (\0 or %00) and other non-printable control characters before performing any validation.

Strictly validate the file type using a whitelist approach, combined with server-side Magic Bytes/File Signature inspection, rather than relying solely on the file extension.

Never use the original filename provided by the user. Generate a randomized, safe filename (e.g., using a UUID) on the server side to store the file securely.

<img width="1719" height="1009" alt="image" src="https://github.com/user-attachments/assets/feb92428-b8ea-4d71-8b43-d69df9fccd1d" />
<img width="1720" height="1012" alt="image" src="https://github.com/user-attachments/assets/3bfe6097-7ee0-4673-b847-2783b31946f4" />




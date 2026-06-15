### 1. Vulnerability Summary

Vulnerability: Source Code Disclosure.

Description: The web server is misconfigured to allow directory listing, and the robots.txt file leaks the location of a /backup directory. Inside, a backup file with a .bak extension is stored. Since the web server does not recognize .bak as an executable script, it returns the file's raw content (source code) instead of executing it.

Impact: High/Critical. Exposure of backend source code often leads to the discovery of proprietary logic, hidden endpoints, and hardcoded credentials (such as database passwords or API keys), paving the way for complete system compromise.

### 2. Exploitation Steps

*Step 1: Access the `/robots.txt` file of the application to identify directories hidden from search engine crawlers.*

*Step 2: Notice the entry Disallow: /backup, indicating a potentially sensitive directory.*

*Step 3: Navigate to GET /backup/. Observe that Directory Listing is enabled, revealing a file named ProductTemplate.java.bak.*

*Step 4: Click on the file or directly request `GET /backup/ProductTemplate.java.bak.`*

*Step 5: The server returns the raw Java source code.*

*Step 6: Analyze the code to extract hardcoded database passwords or connection strings to solve the lab.*

### 3. Root Cause Analysis

Improper Backup Practices: Developers manually created a backup file directly on the server (e.g., via cp file.java file.java.bak or using terminal text editors like Vim/Nano) and forgot to delete it.

Web Server Misconfiguration: Web servers (like Apache/Nginx) are configured to execute specific extensions (e.g., .php, .jsp). Unrecognized extensions like .bak or .swp are treated as plain text (Content-Type: text/plain) and served directly to the client.

Directory Listing Enabled: The server allowed users to view the contents of a directory that lacked an index file.

### 4. Remediation

Use Version Control: Strictly prohibit manual backups on production servers. Use proper version control systems (like Git) for code management.

Restrict File Extensions: Configure the web server to explicitly deny access to sensitive file extensions. Example for Apache:
```
Apache
<FilesMatch "\.(bak|config|sql|swp|env)$">
    Require all denied
</FilesMatch>
Disable Directory Listing: Ensure that directory browsing (autoindex) is disabled globally on the web server.
```

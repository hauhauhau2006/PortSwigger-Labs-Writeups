### 1. Vulnerability Summary

Vulnerability: Information Disclosure (Exposed .git directory).

Description: The application's .git directory is inadvertently exposed to the public internet. This directory contains the entire version control history of the project. By downloading the repository and inspecting previous commits, an attacker can recover sensitive information (such as hardcoded credentials) that a developer previously committed and later attempted to remove.

Impact: Critical. Exposure of the .git folder leads to full source code disclosure. In this specific scenario, it leaked a hardcoded administrator password, allowing an attacker to perform an Account Takeover (ATO) with maximum privileges.

### 2. Exploitation Steps

**Step 1: Discover the exposed .git directory on the target application (often found via directory fuzzing tools or manual testing by navigating to `https://<target-domain>/.git/)`.**

**Step 2: Open a local terminal and use the wget tool to recursively download the entire .git repository from the target server:**
```
Bash
wget -r https://<target-domain>.web-security-academy.net/.git/
```
**Step 3: Navigate into the newly downloaded directory:**
```
Bash
cd <target-domain>.web-security-academy.net/.git
```

**Step 4: Execute the git log command to view the commit history. Search for suspicious commit messages. In this case, a commit by Carlos Montoya was found with the message: "Remove admin password from config".**

**Step 5: Copy the associated commit hash (e.g., ea86bf5ad50939e0a4061578c11533e91f466e39) and use git show to inspect the exact changes made during that commit:**
```
Bash
git show ea86bf5ad50939e0a4061578c11533e91f466e39
```
**Step 6: Analyze the diff output. The red line marked with a minus (-) reveals the original, hardcoded password that was deleted from admin.conf:**

-ADMIN_PASSWORD=am5nxpislnnr20rwon0s

**Step 7: Navigate to the application's login page, authenticate using the username `administrator` and the extracted password `am5nxpislnnr20rwon0s`.**

**Step 8: Access the admin panel and delete the target user `(carlos)` to successfully complete the lab.**

3. Root Cause Analysis
Web Server Misconfiguration: The web server (e.g., Apache, Nginx) was not configured to block access to hidden directories (folders starting with a dot, like .git). Consequently, the server treated the version control files as static assets and served them to any requester.

Poor Secrets Management: The developer temporarily hardcoded a production password directly into the configuration file and committed it to the repository. Even though the developer later changed the code to use environment variables (env('ADMIN_PASSWORD')), the Git architecture retains a permanent historical snapshot of all previous file states.

4. Remediation
Block Access to Hidden Directories: Explicitly configure the web server to deny all external access to .git folders and other hidden files.

For Nginx:

Nginx
location ~ /\.git {
    deny all;
}
For Apache:

Apache
RedirectMatch 404 /\.git
Sanitize Deployment Processes: Never deploy the .git folder to the production web root. Use a robust CI/CD pipeline that builds the application and transfers only the compiled/necessary files to the production environment.

Never Commit Secrets: Passwords, API keys, and cryptographic tokens must never be hardcoded into the source code. Utilize .gitignore to prevent configuration files from being tracked, and rely on secure vault services or environment variables for secret injection. If a secret is accidentally committed, it must be considered compromised, immediately revoked, and rotated.

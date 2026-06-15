### 1. Vulnerability Summary

Vulnerability: Information Disclosure (Verbose Error Messages).

Description: The application fails to handle unexpected input types gracefully. When submitting a non-integer value to a parameter expecting an integer, the backend crashes and returns a detailed stack trace to the user.

Impact: Medium. While it does not directly expose user data, the stack trace leaks sensitive backend information, specifically the underlying Framework and its exact Version. Attackers can leverage this intelligence to search for known vulnerabilities (CVEs) associated with that specific version.

### 2. Exploitation Steps

*Step 1: Intercept the request to view a product using Burp Suite or observe the `URL: GET /product?productId=1.`*

*Step 2: Send the request to Burp Repeater for fuzzing.*

*Step 3: Manipulate the productId parameter by injecting an unexpected string value (Type Mismatch).*

`Payload: GET /product?productId=belphegor HTTP/2`

*Step 4: Send the request and analyze the response.*

*Step 5: The server responds with a 500 Internal Server Error. Scroll through the response body to locate the verbose stack trace.*

*Step 6: Extract the framework name and version number from the error log to submit as the solution.*

### 3. Root Cause Analysis

Type Mismatch & Missing Global Exception Handler: The backend parser attempts to cast the string 'belphegor' to an integer, causing a runtime exception. Because there is no global error handler catching this exception, the system falls back to default behavior.

Debug Mode Enabled: The application was deployed to the production environment with 'Debug Mode' enabled, causing detailed system errors to be rendered on the client side instead of logging them internally.

### 4. Remediation

Disable Debug Mode: Ensure that all debugging features (e.g., display_errors, Debug=True) are disabled in the production environment.

Implement Custom Error Pages: Use try-catch blocks and global exception handling to catch unexpected errors. Serve generic error messages to end-users (e.g., "An unexpected error occurred") and log the actual stack traces securely on the server.

Input Validation: Implement strict input validation to ensure parameters meet the expected data type before processing.

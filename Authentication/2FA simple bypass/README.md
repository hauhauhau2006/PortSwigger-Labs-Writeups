### Lab: 2FA simple bypass

## 1 Vulnerability

**In this lab, we will access to web by bypass**

**It provides me two accounts, use one to log in and another to attack**

**Logic: An attacker with a victim's credentials can bypass the 2FA requirement by navigating directly to the account page**

## 2 Exploitation

**Firstly, I access  wiener account, simultaneously I complete the 2FA challenge**


**After that, I gain the URL**


![Intruder Results](Screenshot%202026-03-02%20123511.png)


**We continue to use the victim Carlos'account. However, we will access without user's 2FA verification code**



**We need to replace URL of carlos account by URL of wiener we found**



**Lab is done**

## 3 Remediation

**Any request to /my-account must check if both authentication factors have been completed**

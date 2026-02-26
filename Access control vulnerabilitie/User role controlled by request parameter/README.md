### Lab : User role controlled by request parameter

## Vulnerability:

**This lab is vertical privilege escalation type**

*Use cookie from user to determine the user's previlege*

*User can modify cookie, so they can manipulate their row*

## Exploitation:

*This lab provides a user account*


*We use it to access then change row*


*Cookie: change admin = false->true*


**We will recive a url contains account carlos user**


**Paste it** 
 ```
GET /admin/delete?username=carlos

```

**Send->Done**




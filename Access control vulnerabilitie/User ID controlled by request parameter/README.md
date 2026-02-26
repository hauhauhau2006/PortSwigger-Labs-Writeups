### Lab: User ID controlled by request parameter(IDOR)

**Log in by account : wiener**

*Replace id = wiener -> carlos*

**Then we have api to fill**

**The server only checks if a session is valid, but fails to check if the session owner is authorized to view id**

![Intruder Results](Screenshot%202026-02-26%20203232.png)

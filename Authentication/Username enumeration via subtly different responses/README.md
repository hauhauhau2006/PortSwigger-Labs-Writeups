### Lab: Username enumeration via subtly different responses

## 1 Vulnerability

**Initialy, I attack as same as previous lab. However, I realize that status code and length all return same response and don't provide me any information**


**Through documentation, I realize that white space is subtle when system shows response**


## 2 Exploitation


**I don't set up like normal, in this lab, I setting up grep-extract at error 'Invalid username or password.'. In this way, I will look for the warning that different with other**

![Intruder Results](Screenshot%202026-03-02%20222947.png)

*I found 'agent' username that have a null space*


**Next step, like previous lab, I attack to find password(replace name account before attacking). Now I look for a password don't return me invalid username or password**


![Intruder Results](Screenshot%202026-03-02%20223444.png)


### Lab is done



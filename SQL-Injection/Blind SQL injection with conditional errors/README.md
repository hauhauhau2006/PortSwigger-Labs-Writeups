# Lab: Blind SQL injection with conditional errors




##  Detection (Nhận diện)
- inject ' into tracking id cookie to return 500 error instead of 200 ok(because this lab is different, we need force to return error to find password)


## Exploitation (Khai thác)

### Payload
- use 'case' funtion and / 0, make sever return error.
```sql
TrackingId=xyz' || (SELECT CASE WHEN (SUBSTR(password,1,1)='a') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || ';
```
**||: oracle operator used to join our malicious subquery with original string**

**SUBSTR(password, position, 1) = 'character' - check if a specific character exists at a specific position(like previous lab, I think password contains 20 string)**

**TO_CHAR(1/0) divide 0 to return 500 status**

**ELSE '' return empty string if guess is wrong,200 ok**

### SOLVE
- attack type: I use cluster bomb to attack 2 different variables
- payload set 1: type "number" from 1-> 20 step : 1 (because length of password is 20)
- payload set 2: simple list a-z,0-9
**After setting up, I find character show 500 and the position on the left*
  

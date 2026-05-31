---
name: sql
description: SQL conventions for queries, schema, and migrations
---
# SQL Language Guidelines

- **Custom Sorting (Database Level):** When sorting categorical strings (like `priority` or `status`) that have a logical order other than alphabetical, prefer using SQL `CASE` statements in the `ORDER BY` clause. This ensures consistency across different parts of the application and offloads sorting logic to the database.
    - Example:
      ```sql
      ORDER BY CASE priority
        WHEN 'Critical' THEN 1
        WHEN 'High'     THEN 2
        WHEN 'Medium'   THEN 3
        WHEN 'Low'      THEN 4
        ELSE 5
      END ASC
      ```

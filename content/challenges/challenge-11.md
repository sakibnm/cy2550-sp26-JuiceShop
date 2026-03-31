# Challenge 11 — Retrieve All User Credentials

| | |
|---|---|
| **Category** | SQL Injection |
| **Difficulty** | ⭐⭐⭐⭐ (4 / 6) |
| **Key Concept** | Full credential theft & defense-in-depth |

---

## Objective

Use SQL Injection to **dump every user's email address and password hash** from the database. This is the capstone challenge — it simulates a full credential theft, one of the most damaging outcomes of a SQL Injection vulnerability.

---

## Scenario

In Challenge 10 you extracted the database schema from `sqlite_master`. You now know the exact structure of the `Users` table — its name and all of its column names. Armed with this intelligence, you can craft a precision UNION SELECT that extracts the most sensitive data in the entire database: user credentials.

In the real world, this is the moment a vulnerability becomes a **data breach**. The attacker has moved from probing and enumeration to actual theft of personal data. If the passwords are weakly hashed (like MD5 without salting), many of them can be reversed using rainbow tables or online hash-lookup services — giving the attacker plaintext passwords that users may have reused across dozens of other sites.

---

## Prerequisites

Before attempting this challenge, you should have completed:

- **Challenge 10** — to know the database schema (table names, column names)
- **Challenge 4** — to understand basic SQL Injection mechanics

From Challenge 10 you should know that:
- The database is **SQLite**
- The user table is called **`Users`**
- It contains columns including `id`, `email`, `password`, and others
- The search endpoint `/rest/products/search?q=` is injectable
- The Products query returns **9 columns**
- Columns **2** and **3** map to the product `name` and `description` fields, which are displayed as text in the UI

---

## Instructions

### Step 1 — Craft the UNION SELECT

1. Open your browser or proxy and navigate to the search endpoint.
2. Using the technique from Challenge 10, craft a UNION SELECT that pulls `email` and `password` from the `Users` table. Place them in columns 2 and 3 (the visible text fields):

   ```
   http://localhost:3000/rest/products/search?q=nonexistent')) UNION SELECT 1,email,password,4,5,6,7,8,9 FROM Users--
   ```

3. The prefix `nonexistent` ensures no real products are returned, so the results contain **only** the injected user data.

### Step 2 — Examine the results

4. Look at the JSON response (or the search results page if viewing in the browser). Each "product" in the results is actually a **user record**:
   - The "name" field contains the user's **email address**
   - The "description" field contains the user's **password hash**

5. **Record all the email/hash pairs.** You should see every user in the database, including:
   - `admin@juice-sh.op`
   - `jim@juice-sh.op`
   - `bender@juice-sh.op`
   - And many others

### Step 3 — Analyze the password hashes

6. Examine the password hashes. They should be **32-character hexadecimal strings** — this is characteristic of **MD5** hashing.

7. Pick a few hashes and look them up in an online hash database:
   - [CrackStation](https://crackstation.net/)
   - [MD5 Online](https://www.md5online.org/)
   - Or search Google directly for the hash string

8. Note which passwords you were able to reverse and what the plaintext values were.

### Step 4 — Verify (optional)

9. Take one of the cracked passwords and try logging in normally (without SQL Injection) using the email and plaintext password. If it works, you have confirmed end-to-end credential theft.

---

## Hints

<details>
<summary>Hint 1 — Column positioning</summary>
The product search results display column 2 as the product name and column 3 as the description. Place <code>email</code> in position 2 and <code>password</code> in position 3 for readable output:

```sql
UNION SELECT 1,email,password,4,5,6,7,8,9 FROM Users
```
</details>

<details>
<summary>Hint 2 — Including the user ID</summary>
To also see the user's ID (useful for identifying the admin, who is typically ID 1), you can place <code>id</code> in one of the other positions:

```sql
UNION SELECT id,email,password,4,5,6,7,8,9 FROM Users
```
</details>

<details>
<summary>Hint 3 — Cracking the admin hash</summary>
The administrator's password hash is <code>0192023a7bbd73250516f069df18b500</code>. This is the MD5 hash of a very common password. Looking it up in any rainbow table or hash database will immediately reveal the plaintext: <code>admin123</code>.
</details>

---

## Questions

1. **How many user accounts did you retrieve?** List at least five email addresses you found in the dump.
2. **Identify the hashing algorithm:**
   - What is the hash length and format?
   - What algorithm produces hashes that look like this?
   - Is this algorithm considered secure for password storage today? Why or why not?
3. **Hash cracking results:**
   - How many of the password hashes were you able to reverse using online lookup tools?
   - What does this tell you about the users' password choices?
   - What does it tell you about the application's password hashing strategy?
4. **Real-world impact** — If this were a real application with real users, describe the full chain of damage:
   - What could an attacker do with the email + plaintext password combinations?
   - What is **credential stuffing**, and why does password reuse make this breach far worse?
   - What legal and regulatory consequences could the company face? (Think: GDPR, breach notification laws)
5. **Defense-in-depth** — This attack succeeded because multiple layers of defense were missing or broken. For each layer below, explain what should have been in place:

   | Layer | What was missing? | What should have been done? |
   |-------|------------------|---------------------------|
   | Query construction | | |
   | Error handling | | |
   | Password hashing | | |
   | Database permissions | | |
   | Network/monitoring | | |
   | Data architecture | | |

---

## Key Concept: The Full SQL Injection Kill Chain

This challenge set has walked you through the same escalation path a real attacker would follow. Here is the complete attack chain:

```
┌─────────────────────────────────────────────────────────────────────┐
│                     THE SQL INJECTION KILL CHAIN                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. RECONNAISSANCE (Challenge 1)                                    │
│     └─▶ Discover the application structure, hidden pages            │
│                                                                     │
│  2. VULNERABILITY DISCOVERY (Challenge 4)                           │
│     └─▶ Find that the login form is injectable                     │
│     └─▶ Confirm with: ' OR 1=1--                                   │
│                                                                     │
│  3. AUTHENTICATION BYPASS (Challenges 4, 6, 7)                     │
│     └─▶ Log in as admin, Jim, Bender without passwords             │
│     └─▶ Access privileged functionality                             │
│                                                                     │
│  4. INJECTION POINT EXPANSION (Challenge 10)                       │
│     └─▶ Discover the search endpoint is also injectable            │
│     └─▶ This endpoint RETURNS data (unlike the login form)         │
│                                                                     │
│  5. SCHEMA ENUMERATION (Challenge 10)                              │
│     └─▶ UNION SELECT from sqlite_master                            │
│     └─▶ Map every table and column in the database                 │
│                                                                     │
│  6. DATA EXFILTRATION (Challenge 11) ◀── YOU ARE HERE              │
│     └─▶ UNION SELECT email, password FROM Users                    │
│     └─▶ All credentials extracted                                   │
│                                                                     │
│  7. OFFLINE CRACKING (post-exfiltration)                           │
│     └─▶ MD5 hashes reversed via rainbow tables                     │
│     └─▶ Plaintext passwords obtained                                │
│                                                                     │
│  8. CREDENTIAL STUFFING (real-world next step)                     │
│     └─▶ Try email + password on Gmail, Amazon, banking, etc.       │
│     └─▶ Users who reused passwords are now compromised everywhere  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**One SQL Injection vulnerability → complete credential theft → cross-site account takeover.** This is why SQL Injection has been in the OWASP Top 10 since its inception and remains one of the most critical web application vulnerabilities.

---

## Key Concept: Why MD5 Is Not Enough

The Juice Shop stores passwords as **unsalted MD5 hashes**. This is catastrophically weak for three reasons:

| Problem | Explanation |
|---------|-------------|
| **MD5 is fast** | Modern GPUs can compute billions of MD5 hashes per second, making brute-force attacks trivial. |
| **No salt** | Without a unique random salt per user, identical passwords produce identical hashes. An attacker can use precomputed rainbow tables to reverse hashes instantly. |
| **Rainbow tables exist** | Massive databases of precomputed MD5 hashes are freely available online. Common passwords can be reversed with a single lookup. |

**What should be used instead:**

| Algorithm | Salted? | Adaptive cost? | Status |
|-----------|---------|---------------|--------|
| MD5 | ❌ | ❌ | **Broken** — never use for passwords |
| SHA-256 | ❌ (usually) | ❌ | **Inadequate** — too fast |
| bcrypt | ✅ | ✅ | **Recommended** |
| scrypt | ✅ | ✅ (memory-hard) | **Recommended** |
| Argon2 | ✅ | ✅ (memory-hard) | **Best practice** (PHC winner) |

Modern password hashing algorithms like **bcrypt**, **scrypt**, and **Argon2** are intentionally slow and include unique salts, making rainbow tables useless and brute-force attacks impractical — even if the hash database is stolen.

---

## Capstone Reflection

You have now completed the full attack chain. Take a moment to reflect on the entire journey:

1. **A single root cause** — lack of parameterized queries — enabled challenges 4, 6, 7, 10, and 11. One fix (prepared statements) would have closed all of them.
2. **Verbose error messages** (Challenge 10) gave you the query structure and database engine for free. Generic errors would have slowed the attack significantly.
3. **Weak password hashing** (this challenge) turned a hash dump into a plaintext password dump. Even if the injection couldn't be prevented, bcrypt/argon2 would have made the stolen hashes practically useless.
4. **Missing access controls** appeared in multiple challenges — the admin panel, the product API, the search endpoint. Least-privilege principles and proper authorization checks add friction at every step.
5. **Client-side-only defenses** (Challenge 8) provided zero security. The server must independently validate everything.

**The lesson is defense-in-depth.** No single control is sufficient. Layer your defenses so that the failure of any one doesn't cascade into a full breach.

---

## Learning Outcome

Experience the **full impact of SQL Injection** — from initial discovery through schema extraction to complete credential theft and offline password cracking. Understand **defense-in-depth**: even if injection is possible, strong password hashing (bcrypt/argon2), proper database permissions, generic error messages, and monitoring can each independently limit the damage. A single vulnerability should never be enough to cause a catastrophic breach — unless every other layer has also failed.

---

[← Back to Index](/challenges/) 

---

## 🎓 Congratulations!

You have completed all 11 challenges in the OWASP Juice Shop XSS & SQL Injection Challenge Set. You now have hands-on experience with:

- **DOM-based, Reflected, and Stored XSS** — three distinct attack vectors with escalating impact
- **SQL Injection** — from basic authentication bypass to full UNION SELECT data exfiltration
- **Client-side vs. server-side security** — and why the client can never be trusted
- **API attack surfaces** — and why backend validation is non-negotiable
- **Defense-in-depth** — layered security controls that limit blast radius even when one layer fails

**Next steps to continue learning:**
- Explore the remaining Juice Shop challenges (there are 100+!)
- Study the [OWASP Top 10](https://owasp.org/www-project-top-ten/) in depth
- Practice with other vulnerable apps: DVWA, WebGoat, HackTheBox
- Read the [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) for defensive techniques

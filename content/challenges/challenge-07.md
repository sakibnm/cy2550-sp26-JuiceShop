# Challenge 7 — Login Bender via SQL Injection

| | |
|---|---|
| **Category** | SQL Injection |
| **Difficulty** | ⭐⭐⭐ (3 / 6) |
| **Key Concept** | Password strength ≠ safety |

---

## Objective

Log in as Bender's user account using SQL Injection. This challenge reinforces the targeted injection technique from Challenge 6 and drives home an important lesson: a strong password does not protect you if the authentication logic is broken.

---

## Scenario

Bender is another registered user on the Juice Shop. Unlike some of the other users, **Bender chose a strong password** — long, complex, and not found in any rainbow table or dictionary. If you managed to extract his password hash (from a later challenge or by other means), you would find that cracking it is not feasible.

But it doesn't matter. The login form has the same SQL Injection vulnerability you've been exploiting. If you can bypass the password check entirely, the strength of Bender's password is irrelevant. The lock on the front door doesn't help if the wall next to it has a hole in it.

---

## Instructions

1. **Find Bender's email address:**
   - Use the same reconnaissance techniques from Challenge 6: product reviews, the admin panel, or educated guessing.
   - Bender is a character from the TV show *Futurama*. His email follows the same `@juice-sh.op` format.

2. **Craft the injection payload:**
   - Navigate to the **Login page** (`/#/login`).
   - In the **Email** field, enter Bender's email address followed by the injection that comments out the password check.
   - Enter **anything** in the Password field.
   - Click **"Log in."**

3. **Verify** you are logged in as Bender by checking the account name or navigating to `/#/profile`.

---

## Hints

<details>
<summary>Hint 1 — Bender's email</summary>
Following the Juice Shop's naming convention, Bender's email address is <code>bender@juice-sh.op</code>.
</details>

<details>
<summary>Hint 2 — The payload</summary>
This is identical to the Challenge 6 technique:

```
bender@juice-sh.op'--
```

The resulting query:

```sql
SELECT * FROM Users
WHERE email = 'bender@juice-sh.op'--' AND password = '...'
```

The password check is commented out. Bender's strong password is never evaluated.
</details>

---

## Questions

1. **Why does a strong password NOT protect Bender from this attack?** Explain the specific mechanism by which the password check is bypassed.
2. Many security awareness programs focus heavily on password strength ("use 12+ characters, mix uppercase, lowercase, numbers, symbols"). Based on what you've learned in Challenges 4, 6, and 7, **what are the limits of password-focused security?** What other defenses are needed?
3. If you were advising the Juice Shop's developers, **what single code change** would fix the login form for ALL users at once — regardless of who they are or how strong their passwords are?
4. **Authentication bypass vs. credential theft** — define each:
   - **Authentication bypass:** ___
   - **Credential theft:** ___
   - Which one did you perform in this challenge?
   - Which one would you perform in Challenge 11 (Retrieve All User Credentials)?
5. Even if the SQL Injection were fixed, what additional authentication mechanism would add a layer of defense so that stolen or bypassed passwords alone are not enough? (Hint: it's a three-letter abbreviation.)

---

## Key Concept: Defense-in-Depth vs. Single Points of Failure

This challenge illustrates why **defense-in-depth** matters. Consider these layers:

| Layer | Control | Does it help against SQLi? |
|-------|---------|---------------------------|
| 1 | Strong password policy | ❌ No — the password is never checked |
| 2 | Parameterized queries | ✅ **Yes — eliminates SQLi entirely** |
| 3 | Multi-factor authentication (MFA) | ✅ Yes — even if login is bypassed, second factor is needed |
| 4 | Web Application Firewall (WAF) | ⚠️ Partial — can detect common patterns but can be evaded |
| 5 | Account anomaly detection | ⚠️ Partial — might flag unusual login patterns |
| 6 | Rate limiting | ❌ No — SQLi bypass works on the first try |

If the Juice Shop relied **only** on password strength (Layer 1), every user is vulnerable. The fix is at Layer 2 (parameterized queries), with Layers 3–5 as additional safety nets.

**The lesson:** No single security control is sufficient. A strong password is good, but it's one layer among many — and it's useless if the layer in front of it (the query logic) is broken.

---

## Learning Outcome

Reinforce **targeted SQL Injection** skills and understand that **password strength is irrelevant when the authentication logic itself is broken**. The real defense against SQL Injection is fixing the code (parameterized queries), not making users choose better passwords. This challenge also introduces the concept of **defense-in-depth** — layering multiple security controls so that the failure of any single one doesn't compromise the system.

---

[← Back to Index](/challenges/) 

+++
date = '2026-03-31T14:01:43-04:00'
draft = false
title = 'Challenges'
+++
# OWASP Juice Shop — XSS & SQL Injection Challenge Set

> **Prerequisites:** A running instance of OWASP Juice Shop (Docker, Node.js, or cloud-hosted). A modern browser with Developer Tools. Optionally, a proxy tool such as Burp Suite Community Edition or OWASP ZAP.

---

## Challenge Index

| #  | Challenge | Category | Difficulty | Key Concept |
|----|-----------|----------|------------|-------------|
| 1  | [Find the Score Board](challenge-01-score-board.md) | Recon | ⭐ | Client-side code inspection |
| 2  | [DOM XSS](challenge-02-dom-xss.md) | XSS | ⭐ | DOM-based script injection |
| 3  | [Bonus Payload](challenge-03-bonus-payload.md) | XSS | ⭐ | Real-world XSS impact |
| 4  | [Login Admin — SQLi](challenge-04-login-admin.md) | SQLi | ⭐⭐ | Authentication bypass |
| 5  | [Reflected XSS](challenge-05-reflected-xss.md) | XSS | ⭐⭐ | URL-based reflected injection |
| 6  | [Login Jim — SQLi](challenge-06-login-jim.md) | SQLi | ⭐⭐⭐ | Targeted injection |
| 7  | [Login Bender — SQLi](challenge-07-login-bender.md) | SQLi | ⭐⭐⭐ | Password strength ≠ safety |
| 8  | [Client-side XSS Protection Bypass](challenge-08-client-side-bypass.md) | XSS | ⭐⭐⭐ | Client validation is not security |
| 9  | [API-only Persisted XSS](challenge-09-api-xss.md) | XSS | ⭐⭐⭐ | API attack surface, stored XSS |
| 10 | [Exfiltrate DB Schema](challenge-10-db-schema.md) | SQLi | ⭐⭐⭐ | UNION SELECT, error disclosure |
| 11 | [Retrieve All User Credentials](challenge-11-credential-theft.md) | SQLi | ⭐⭐⭐⭐ | Full credential theft, defense-in-depth |

---

## Progression Map

```
  ⭐ Beginner                ⭐⭐ Easy              ⭐⭐⭐ Intermediate           ⭐⭐⭐⭐ Hard
┌──────────────┐        ┌──────────────┐       ┌────────────────────┐       ┌──────────────────┐
│ 1. Score     │───────▶│ 4. Login     │──────▶│ 6. Login Jim (SQLi)│──────▶│ 11. Credential   │
│    Board     │        │    Admin     │       │ 7. Login Bender    │       │     Theft (SQLi) │
└──────────────┘        │    (SQLi)    │       │ 10. DB Schema      │       └──────────────────┘
┌──────────────┐        └──────────────┘       └────────────────────┘
│ 2. DOM XSS   │───────▶┌──────────────┐       ┌────────────────────┐
│ 3. Bonus     │        │ 5. Reflected │──────▶│ 8. Client-side     │
│    Payload   │        │    XSS       │       │    Bypass (XSS)    │
└──────────────┘        └──────────────┘       │ 9. API XSS         │
                                               └────────────────────┘
```

---

## How to Use This Material

1. Work through the challenges in order — each builds on skills from the previous ones.
2. Read the **Scenario** to understand the context before attempting anything.
3. Try to solve the challenge using only the **Instructions** first.
4. If stuck, reveal the **Hints** one at a time.
5. After solving, answer all **Questions** — these reinforce the learning.
6. Read the **Learning Outcome** to confirm you understood the core concept.

---

## References

- [OWASP Juice Shop Official Site](https://owasp.org/www-project-juice-shop/)
- [Pwning OWASP Juice Shop (Companion Guide)](https://pwning.owasp-juice.shop/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

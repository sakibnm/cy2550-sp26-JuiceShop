# Challenge 8 — Client-side XSS Protection Bypass

| | |
|---|---|
| **Category** | Cross-Site Scripting (XSS) |
| **Difficulty** | ⭐⭐⭐ (3 / 6) |
| **Key Concept** | Client-side validation is not security |

---

## Objective

Perform a **persisted (stored) XSS** attack by bypassing the client-side input validation on the registration form. The payload you inject will be saved to the database and executed every time an admin views the user list.

---

## Scenario

The Juice Shop's user registration form includes client-side JavaScript validation that checks whether the email field contains a properly formatted email address. If you type something that isn't a valid email, the form won't let you submit.

This might seem like it prevents XSS — after all, `<iframe src="javascript:alert('xss')">` is not a valid email address, so the form rejects it. But here's the critical lesson: **client-side validation runs in YOUR browser**. You control your browser. You can bypass, disable, or modify any client-side check.

If the server does not perform its own validation, any payload you sneak past the browser will be accepted, stored in the database, and served to other users.

---

## Instructions

1. Open the **Registration page** (`/#/register`).
2. Fill in the form with valid-looking data:
   - **Email:** any valid email (e.g., `test@test.com`)
   - **Password:** any password (e.g., `password123`)
   - **Repeat Password:** same as above
   - **Security Question:** choose any, and provide an answer
3. **Before clicking "Register,"** set up your interception:

   **Option A — Browser Developer Tools:**
   - Open Developer Tools → **Network** tab.
   - Submit the form.
   - Find the registration request (POST to `/api/Users/`).
   - Right-click → **Edit and Resend** (Firefox) or **Copy as cURL** (Chrome) and modify in terminal.
   - Change the `email` field in the request body to:
     ```
     <iframe src="javascript:alert(`xss`)">
     ```
   - Resend the modified request.

   **Option B — Burp Suite / ZAP:**
   - Enable **Intercept** in your proxy.
   - Submit the form.
   - When the request is caught, modify the `email` field in the JSON body to:
     ```
     <iframe src="javascript:alert(`xss`)">
     ```
   - Forward the modified request.

4. The server should accept the request and create the user with the XSS payload as the email.
5. **Trigger the payload:**
   - Log in as the **administrator** (use the SQLi technique from Challenge 4).
   - Navigate to the **Administration page** (`/#/administration`).
   - The page displays all registered users. When the malicious "email" is rendered, the XSS payload will execute.

---

## Hints

<details>
<summary>Hint 1 — Understanding the bypass</summary>
Client-side validation is JavaScript running in your browser. It checks the email format before the form submits. But the actual HTTP request is just JSON sent to an API endpoint. You can craft that request yourself without ever using the form.
</details>

<details>
<summary>Hint 2 — Using curl</summary>
If you prefer the command line:

```bash
curl -X POST http://localhost:3000/api/Users/ \
  -H "Content-Type: application/json" \
  -d '{"email":"<iframe src=\"javascript:alert(`xss`)\">","password":"test123","passwordRepeat":"test123","securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"...","updatedAt":"..."},"securityAnswer":"test"}'
```

Adjust the URL and security question fields as needed for your instance.
</details>

<details>
<summary>Hint 3 — Verifying the stored payload</summary>
If the alert doesn't trigger on the Administration page, check the API response from your registration request. If the server returned a 201 Created with the XSS payload in the email field, the payload is stored. It will execute when any page renders that email without encoding.
</details>

---

## Questions

1. **Why did the client-side validation not stop you?** Describe the exact mechanism you used to bypass it.
2. The payload is now **stored in the database**. Compare this to the previous XSS types you've encountered:

   | | DOM XSS | Reflected XSS | Stored XSS (this) |
   |---|---------|--------------|-------------------|
   | Payload persisted? | | | |
   | Requires victim to click a link? | | | |
   | How many users affected? | | | |

   Fill in the table and explain why stored XSS is generally considered the most dangerous type.

3. **"Never trust the client"** — explain this security principle in your own words. Give two other examples (beyond this challenge) where relying on client-side validation could lead to security issues.
4. How should the server validate the email field? Name at least two server-side controls:
   - One for **input validation** (preventing bad data from being stored)
   - One for **output encoding** (preventing stored data from executing as code)
5. If you were the developer, would you **remove** the client-side validation? Why or why not? What purpose does it serve even though it's not a security control?

---

## Key Concept: Client-Side Validation vs. Server-Side Security

```
┌─────────────────────────────────────────────────────────────┐
│                      THE BROWSER                            │
│                                                             │
│  ┌──────────────────────┐                                   │
│  │  Registration Form   │                                   │
│  │                      │     ┌─────────────────────┐       │
│  │  Email: [test@t.com] │────▶│ Client-side JS      │       │
│  │  Pass:  [••••••••]   │     │ "Is this a valid    │       │
│  │                      │     │  email format?"     │       │
│  │  [Register]          │     │                     │       │
│  └──────────────────────┘     │  ✅ Yes → submit    │       │
│                               │  ❌ No  → block     │       │
│                               └─────────────────────┘       │
│                                        │                    │
│   ATTACKER BYPASSES ──────────────────▶│ (intercepts here)  │
│   by modifying the HTTP request        │                    │
│   after validation but before sending  │                    │
└────────────────────────────────────────│────────────────────┘
                                         │
                                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      THE SERVER                             │
│                                                             │
│   ❌ No server-side validation!                             │
│   Payload accepted and stored in database.                  │
│                                                             │
│   Later: Admin views user list → payload renders → XSS!    │
└─────────────────────────────────────────────────────────────┘
```

**Client-side validation** is a **user experience** feature. It gives users instant feedback ("that's not a valid email") without a round trip to the server. It is NOT a security control because the user controls the client.

**Server-side validation** is the **security** control. It must independently verify all input regardless of what the client did or didn't check.

**Both are needed.** Client-side for UX; server-side for security. Never one without the other.

---

## Learning Outcome

Understand that **client-side validation is a user experience feature, not a security control**. Any check that runs in the browser can be bypassed by intercepting and modifying the HTTP request. Learn the critical importance of **server-side input validation** and **output encoding** — the server must never trust that the client has done its job.

---

[← Back to Index](/challenges/) 

# Challenge 9 — API-only Persisted XSS

| | |
|---|---|
| **Category** | Cross-Site Scripting (XSS) |
| **Difficulty** | ⭐⭐⭐ (3 / 6) |
| **Key Concept** | API attack surface & stored XSS |

---

## Objective

Inject a persisted XSS payload into a product description by **directly calling the Juice Shop's REST API** — without using the frontend application at all. The payload will be stored in the database and execute for every user who views the affected product.

---

## Scenario

In Challenge 8 you bypassed client-side validation on a form. But what if there's no form at all? Many modern web applications expose **REST APIs** that power the frontend. These APIs are often more permissive than the UI suggests — they may accept fields the frontend never sends, allow methods the UI never uses (like PUT or DELETE), and skip validation that only exists in the frontend JavaScript.

The Juice Shop's product API allows modification of product data via PUT requests. The frontend never exposes an "edit product" form to regular users, so a developer might assume the data is safe. But an attacker doesn't need the frontend — they can call the API directly using tools like `curl`, Postman, or Burp Suite.

If the API accepts HTML/JavaScript in a product description and the frontend renders that description without encoding, you have a **stored XSS** vulnerability that affects every visitor who views the product page.

---

## Instructions

1. **Discover the API endpoint:**
   - Open Developer Tools → **Network** tab.
   - Browse the Juice Shop's product pages and observe the API calls.
   - You should see requests to `/api/Products/` and `/api/Products/<ID>`.
   - Note: a GET request to `/api/Products/` returns all products with their IDs.

2. **Choose a target product:**
   - Pick any product and note its numeric **ID** (e.g., `1` for Apple Juice).

3. **Send the malicious PUT request:**

   Using **curl**:
   ```bash
   curl -X PUT http://localhost:3000/api/Products/1 \
     -H "Content-Type: application/json" \
     -d '{"description": "<iframe src=\"javascript:alert(`xss`)\">"}'
   ```

   Using **Postman**:
   - Method: `PUT`
   - URL: `http://localhost:3000/api/Products/1`
   - Headers: `Content-Type: application/json`
   - Body (raw JSON):
     ```json
     {"description": "<iframe src=\"javascript:alert(`xss`)\">"}
     ```

   Using **Burp Suite Repeater**:
   - Craft a PUT request to `/api/Products/1` with the JSON body above.
   - Send it and check for a `200 OK` response.

4. **Trigger the payload:**
   - Navigate to the Juice Shop's main page in your browser.
   - Find the product you modified. Click on it or view its details.
   - The XSS alert should pop up when the description is rendered.

---

## Hints

<details>
<summary>Hint 1 — No authentication required</summary>
The Juice Shop's product API does not require authentication for PUT requests — that's actually a separate vulnerability (Broken Access Control). For this challenge, it means you can modify any product without logging in.
</details>

<details>
<summary>Hint 2 — Check the response</summary>
After sending the PUT request, check the JSON response. If the <code>description</code> field contains your payload, the injection was stored successfully. If the field was sanitized or stripped, the server has some protection and you may need to try a different field or encoding.
</details>

<details>
<summary>Hint 3 — Alternative approach</summary>
If the description field doesn't work, try other writable fields on the product. Some APIs accept more fields than the documentation suggests. Check what fields are returned in a GET response — any of them might be writable.
</details>

---

## Questions

1. **The frontend never shows a "edit product" form.** Yet you were able to modify product data. What security assumption did the developers make, and why was it wrong?
2. **No authentication was required** for the PUT request. This is actually two vulnerabilities in one:
   - Vulnerability 1: ___ (related to access control)
   - Vulnerability 2: ___ (related to input handling)
   
   Name both and explain how they compound each other.
3. **Stored XSS blast radius:** In Challenge 5 (reflected XSS), the victim had to click a crafted link. In this challenge, how many users are affected? Does the attacker need to interact with each victim individually?
4. The frontend might sanitize product descriptions before displaying them, but the **API** doesn't sanitize input before storing it. Why is this a problem even if the current frontend is safe? (Hint: think about other consumers of the API — mobile apps, partner integrations, future frontend redesigns.)
5. **Output encoding** is often cited as the primary defense against stored XSS. Explain the difference between these two approaches and why defense experts recommend both:
   - **Input validation:** Reject or strip dangerous characters when data is received.
   - **Output encoding:** Escape special characters when data is rendered in HTML.

---

## Key Concept: APIs Are an Independent Attack Surface

```
┌──────────────────────────────────────────────────┐
│              WHAT DEVELOPERS ASSUME               │
│                                                   │
│   Users ──▶ Frontend UI ──▶ API ──▶ Database     │
│             (validates)                           │
│                                                   │
│   "The frontend validates input, so the           │
│    API doesn't need to."                          │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│              WHAT ACTUALLY HAPPENS                 │
│                                                   │
│   Users ──▶ Frontend UI ──▶ API ──▶ Database     │
│                              ▲                    │
│   Attackers ─── curl ────────┘                    │
│   Attackers ─── Postman ─────┘                    │
│   Attackers ─── Burp Suite ──┘                    │
│   Mobile apps ───────────────┘                    │
│   Partner integrations ──────┘                    │
│                                                   │
│   The API is accessible to ANYONE, not just       │
│   the frontend. It must defend itself.            │
└──────────────────────────────────────────────────┘
```

**Key rules for API security:**

1. **Validate all input at the API layer** — never assume the frontend did it.
2. **Enforce authentication and authorization** on every endpoint and method (GET, POST, PUT, DELETE).
3. **Apply the principle of least privilege** — don't allow PUT/DELETE on resources unless explicitly required.
4. **Encode output** wherever data is rendered — don't trust that stored data is clean just because "our frontend validated it."

---

## Comparison: All XSS Types Encountered So Far

| Challenge | XSS Type | Payload Location | Persisted? | Victims | Tool Needed |
|-----------|----------|-----------------|------------|---------|-------------|
| 2 — DOM XSS | DOM-based | Search bar → DOM | No | Self only | Browser |
| 3 — Bonus | DOM-based | Search bar → DOM | No | Self only | Browser |
| 5 — Reflected | Reflected | URL param → response | No | Link clickers | Browser |
| 8 — Client bypass | Stored | Registration → DB → admin page | Yes | Admins | Proxy/DevTools |
| **9 — API XSS** | **Stored** | **API → DB → product page** | **Yes** | **All visitors** | **curl/Postman** |

Notice how the attack surface and blast radius increase as we move down the table. Stored XSS via the API is the most dangerous: it requires no victim interaction, persists indefinitely, and affects everyone who views the page.

---

## Learning Outcome

Learn that **APIs are an independent attack surface** that must enforce their own security controls — authentication, authorization, input validation, and output encoding. Understand that **stored/persisted XSS** injected via an API has the widest blast radius of any XSS type: it affects every user who views the affected page, requires no special link or social engineering, and persists until the malicious data is removed from the database.

---

[← Back to Index](index.md) | [Previous: Challenge 8](challenge-08-client-side-bypass.md) | [Next: Challenge 10 — Exfiltrate DB Schema →](challenge-10-db-schema.md)

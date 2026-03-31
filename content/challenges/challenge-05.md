# Challenge 5 — Reflected XSS

| | |
|---|---|
| **Category** | Cross-Site Scripting (XSS) |
| **Difficulty** | ⭐⭐ (2 / 6) |
| **Key Concept** | URL-based reflected injection |

---

## Objective

Perform a reflected XSS attack by injecting a malicious payload into a URL parameter that the server echoes back into the page without sanitization.

---

## Scenario

After placing an order in the Juice Shop, you are shown an order confirmation or tracking page. The **order tracking ID** displayed on this page is pulled directly from a URL parameter and rendered into the HTML. The server takes the value from the URL, includes it in its response, and the browser renders it — no sanitization, no encoding.

This means an attacker can craft a malicious URL containing JavaScript, send it to a victim (via email, chat, or social media), and the script will execute the moment the victim clicks the link. The victim's browser trusts the page because it comes from the legitimate Juice Shop domain.

---

## Instructions

1. **Place an order** in the Juice Shop:
   - Add any item to your basket.
   - Go to the basket and proceed through checkout.
   - Complete the order.
2. After the order is confirmed, navigate to the **Order Tracking** page. You can find it via the navigation menu or at `/#/track-result`.
3. Examine the URL in your browser's address bar. Look for a parameter that controls the tracking ID displayed on the page (e.g., `?id=...`).
4. Replace the tracking ID value in the URL with the following payload:

   ```
   <iframe src="javascript:alert(`xss`)">
   ```

5. Load the modified URL (press Enter in the address bar).
6. You should see the alert dialog pop up, confirming the reflected XSS.

---

## Hints

<details>
<summary>Hint 1</summary>
The Order Tracking feature can also be accessed directly. Try navigating to <code>/#/track-result?id=test</code> and see if the word "test" appears on the page. If it does, that parameter is injectable.
</details>

<details>
<summary>Hint 2</summary>
If the <code>id</code> parameter doesn't work, look at the network requests in Developer Tools after placing an order. The tracking result endpoint might use a different parameter name or URL structure.
</details>

<details>
<summary>Hint 3</summary>
The Juice Shop may also reflect input in other places. Check the order confirmation/tracking email or the <code>/track-result/new</code> endpoint for reflected parameters.
</details>

---

## Questions

1. **How is reflected XSS different from the DOM XSS** you performed in Challenge 2?
   - In DOM XSS, which component handles the malicious input — the server or the browser?
   - In reflected XSS, where does the payload travel before it reaches the victim's browser?
2. Imagine you send the following link to a colleague: `https://juice-shop.example.com/#/track-result?id=<iframe src="javascript:alert('xss')">`. Walk through exactly what happens when they click it.
3. Why is this called **"reflected"** XSS? What is being reflected, and from where to where?
4. Could this attack steal the victim's session cookie? If so, how would you modify the payload? (Describe the approach — do not build a working exploit.)
5. Name two defenses that could mitigate reflected XSS:
   - One **server-side** defense
   - One **browser/HTTP-header** defense

---

## Key Concept: Reflected XSS

In a reflected XSS attack, the malicious payload takes this path:

```
Attacker crafts URL with payload
        │
        ▼
Victim clicks the link
        │
        ▼
Browser sends request to server (payload in URL/parameter)
        │
        ▼
Server includes payload in the HTML response (unsanitized)
        │
        ▼
Browser renders the response → payload executes as JavaScript
```

The payload is **"reflected"** off the server — it goes in with the request and comes back in the response. The server doesn't store it; it's a one-shot attack that requires the victim to click a crafted link.

**Comparison of XSS types so far:**

| Type | Payload stored? | Server involved? | Requires victim action? |
|------|----------------|-----------------|------------------------|
| **DOM XSS** (Challenge 2) | No | No — purely client-side | Yes — must trigger the JS |
| **Reflected XSS** (this challenge) | No | Yes — server reflects it | Yes — must click crafted URL |
| **Stored XSS** (Challenges 8 & 9) | Yes — in database | Yes — serves it to all visitors | No — just viewing the page |

---

## Learning Outcome

Understand **reflected XSS** — where malicious input is sent to the server via a URL or request parameter and reflected back in the response without sanitization. This is dangerous in **phishing and social engineering** scenarios because the malicious URL points to the legitimate domain, making it more convincing. Defenses include **output encoding** on the server side and the **Content-Security-Policy** HTTP header on the browser side.

---

[← Back to Index](index.md) | [Previous: Challenge 4](challenge-04-login-admin.md) | [Next: Challenge 6 — Login Jim via SQLi →](challenge-06-login-jim.md)

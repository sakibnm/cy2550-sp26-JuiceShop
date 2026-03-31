# Challenge 2 — DOM XSS

| | |
|---|---|
| **Category** | Cross-Site Scripting (XSS) |
| **Difficulty** | ⭐ (1 / 6) |
| **Key Concept** | DOM-based script injection |

---

## Objective

Execute a DOM-based XSS attack using the Juice Shop's search bar.

---

## Scenario

The Juice Shop has a product search feature. When you type a search term, the application takes your input and inserts it directly into the page's DOM using client-side JavaScript — without sanitizing it first. This means that if you type HTML or JavaScript instead of a product name, the browser will interpret and execute it as real code.

Your goal is to prove this vulnerability exists by making the browser execute a script you control.

---

## Instructions

1. Navigate to the Juice Shop's main page and locate the **search bar** (magnifying glass icon in the top navigation).
2. Click the search bar and enter the following payload exactly as written:

   ```
   <iframe src="javascript:alert(`xss`)">
   ```

3. Press **Enter** and observe what happens on the page.
4. You should see a JavaScript alert dialog pop up with the text "xss."
5. Open **Developer Tools → Elements** tab and find where your payload was inserted into the DOM.

---

## Hints

<details>
<summary>Hint 1</summary>
The search bar is the most obvious input field on the page. The application renders your search term directly into the results heading (e.g., "Search Results — <your input>").
</details>

<details>
<summary>Hint 2</summary>
An <code>&lt;iframe&gt;</code> with a <code>javascript:</code> source causes the browser to execute the JavaScript expression when it renders the element. This is an alternative to <code>&lt;script&gt;</code> tags, which some basic filters block.
</details>

---

## Questions

1. Did the payload execute? What exactly did you see on screen?
2. Why does an `<iframe>` with a `javascript:` source trigger script execution? How is this different from using a `<script>` tag?
3. Open Developer Tools → Elements. Where exactly was your payload inserted into the DOM? What HTML element contains it?
4. This is called **DOM-based** XSS. The server never saw your malicious payload — it was all handled by client-side JavaScript. How is this different from the server reflecting your input in the HTTP response?
5. As a developer, what JavaScript API or technique would you use to safely insert user input into the DOM? (Hint: think about `innerText` vs. `innerHTML`.)

---

## Key Concept: DOM-Based XSS

In a DOM-based XSS attack, the vulnerability exists entirely in **client-side JavaScript**. The flow is:

```
User input → Client-side JS reads input → JS writes input to DOM (unsanitized) → Browser renders it as HTML/JS
```

The server is never involved in the injection. The malicious payload never appears in an HTTP response — it lives only in the browser. This makes DOM XSS harder to detect with server-side security tools like Web Application Firewalls (WAFs).

---

## Learning Outcome

Understand **DOM-based XSS** — where the vulnerability exists entirely in client-side JavaScript that takes user input and writes it to the page without sanitization. The fix is to use safe DOM APIs (like `textContent` or `innerText`) instead of `innerHTML`, or to sanitize input before rendering.

---

[← Back to Index](index.md) | [Previous: Challenge 1](challenge-01-score-board.md) | [Next: Challenge 3 — Bonus Payload →](challenge-03-bonus-payload.md)

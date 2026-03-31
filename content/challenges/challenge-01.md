# Challenge 1 — Find the Score Board

| | |
|---|---|
| **Category** | Recon / Security Misconfiguration |
| **Difficulty** | ⭐ (1 / 6) |
| **Key Concept** | Client-side code inspection |

---

## Objective

Locate the hidden Score Board page of the Juice Shop.

---

## Scenario

Every good hacker starts with reconnaissance. The Juice Shop tracks your progress on a hidden Score Board page, but there is no visible link to it anywhere in the navigation menu. The developers commented it out — but they forgot that everything shipped to the browser is visible to the user. Your first task is to find it.

---

## Instructions

1. Open the Juice Shop in your browser and explore the application normally. Click through the navigation menu, footer links, and any other visible UI elements.
2. Open your browser's **Developer Tools** (F12 or right-click → Inspect).
3. Examine the page source, JavaScript bundles, or route definitions to discover hidden paths. Look for:
   - Commented-out HTML in the page source.
   - Route definitions in the Angular/JavaScript files.
   - References to paths that don't appear in the visible navigation.
4. Once you find the path, navigate to it directly in the browser's address bar.
5. Confirm the Score Board page loads and shows the full list of challenges.

---

## What to Look For

- How does the Angular frontend define its routes? Where are those route definitions stored?
- Are there commented-out `<li>` or `<a>` elements in the HTML that reveal hidden pages?
- Does the main JavaScript bundle contain a list of all application routes?

---

## Questions

1. Where exactly did you find the reference to the Score Board? (HTML source? JavaScript file? Network traffic?)
2. What is the exact URL path of the Score Board?
3. Did you discover any other hidden or interesting routes while searching?
4. Why is shipping client-side route definitions a security concern — even if the link is removed from the visible UI?
5. How could the developers have properly restricted access to the Score Board if they intended it to be hidden?

---

## Learning Outcome

Understand that **client-side code is never truly hidden** from the user. Any route, feature, or configuration shipped to the browser can be discovered through inspection. Security through obscurity — hiding a link but leaving the page accessible — is not a valid security control.

---

[← Back to Index](index.md) | [Next: Challenge 2 — DOM XSS →](challenge-02-dom-xss.md)

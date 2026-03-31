# Challenge 3 — Bonus Payload

| | |
|---|---|
| **Category** | Cross-Site Scripting (XSS) |
| **Difficulty** | ⭐ (1 / 6) |
| **Key Concept** | Real-world XSS impact |

---

## Objective

Embed a richer XSS payload in the same DOM XSS vulnerability to demonstrate that the impact of XSS goes far beyond simple alert boxes.

---

## Scenario

In the previous challenge you proved that the search bar is injectable — you made the browser execute `alert('xss')`. A common misconception is that XSS is just about pop-up boxes. In reality, anything that can be expressed as HTML or JavaScript can be injected: fake login forms, hidden keyloggers, cryptocurrency miners, embedded media, redirects to phishing sites, and more.

To drive this point home, you will now inject an entire **embedded audio player** into the Juice Shop's search results page. If an attacker can embed a media player, they can embed anything.

---

## Instructions

1. Open the Juice Shop and click on the **search bar**.
2. Enter the following payload — it is a full SoundCloud embed widget:

   ```html
   <iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https://api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>
   ```

3. Press **Enter**.
4. Observe the search results page. An audio player should appear, and music may begin playing automatically.

---

## Hints

<details>
<summary>Hint 1</summary>
This uses the exact same vulnerability as Challenge 2. The only difference is the payload is more complex HTML — the injection mechanism is identical.
</details>

<details>
<summary>Hint 2</summary>
If the audio doesn't autoplay, your browser might be blocking it due to autoplay policies. The player widget should still render visually even if sound is muted.
</details>

---

## Questions

1. What appeared on the page? Did you see an audio player? Did audio play?
2. The payload was a benign music player. Now imagine replacing it with each of the following. What would the impact be?
   - A fake login form styled to look like the Juice Shop's real login page
   - A hidden `<script>` tag that sends the user's cookies to an attacker's server
   - A `<script>` tag that loads an external cryptocurrency miner
   - An invisible `<iframe>` that silently performs actions on another site the user is logged into (CSRF)
3. Why do security demonstrations typically use `alert()` boxes even though the real impact is much worse?
4. As a developer, what is the difference between **input sanitization** (cleaning data on the way in) and **output encoding** (escaping data on the way out)? Which one prevents this attack, and why?

---

## Key Concept: XSS Impact Is Not Just `alert()`

Security researchers use `alert()` as a **proof of concept** — it's a harmless, visible way to prove that arbitrary JavaScript can run. But once you can execute JavaScript in a victim's browser, you can:

| Attack | Technique |
|--------|-----------|
| **Steal session cookies** | `document.cookie` sent to attacker's server |
| **Keylogging** | Event listeners on all input fields |
| **Phishing** | Inject a fake login form over the real page |
| **Defacement** | Rewrite the page content |
| **Cryptojacking** | Load a hidden mining script |
| **Worm propagation** | XSS payload that reposts itself (e.g., Samy worm) |

The `alert()` box is the canary — the real damage comes after.

---

## Learning Outcome

Recognize that **XSS impact goes far beyond `alert()` boxes**. Arbitrary HTML injection can be used to deface pages, steal credentials via fake login forms, hijack sessions, or execute complex JavaScript payloads. Any proof-of-concept `alert()` should be treated as evidence that a much more serious attack is possible.

---

[← Back to Index](/challenges/) 

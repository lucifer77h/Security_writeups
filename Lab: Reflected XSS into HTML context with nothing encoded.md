# Reflected XSS into HTML context with nothing encoded

**Platform:** PortSwigger Web Security Academy
**Topic:** Cross-Site Scripting (XSS)
**Difficulty:** Apprentice

## Objective
Exploit a reflected XSS vulnerability where user input is reflected into the HTML body with no encoding, and trigger `alert(1)`.

## Recon
The lab has a search function. The search term gets echoed back on the results page (e.g. "0 results for 'test'"), placed directly into the HTML body with no encoding.

## Payload
```
<script>alert('XSS')</script>
```

Entered into the search box and submitted. Since the input lands directly in the HTML body (not inside an attribute), the browser renders the `<script>` tag as real markup and executes it.

## Vulnerability
Reflected XSS — user input is inserted into the HTML response without any encoding or sanitization, so any HTML/JS in the input is executed by the browser.

## Fix
HTML-encode all reflected user input (`<` → `&lt;`, `>` → `&gt;`) before placing it in the response, so it's rendered as text, not as live markup.

## Takeaway
When input lands directly in the HTML body (not inside a tag or attribute), no escape sequence is needed — a plain `<script>` tag is enough to prove the vulnerability.

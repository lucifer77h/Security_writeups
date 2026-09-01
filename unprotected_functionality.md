# Lab 3: Unprotected Functionality (Client-Side Disclosure)

**Source:** PortSwigger Web Security Academy — Access Control
**Difficulty:** Apprentice
**Date:** 31 Aug 2026
**Tool:** Burp Suite

---

## Objective
Find the hidden administrator panel and delete the user `carlos`.

## Recon / Observations

**Attempt 1 — robots.txt**
- Checked `/robots.txt` first, same technique that worked on Lab 2.
- Returned nothing useful this time — this app doesn't disclose the
  admin path via robots.txt, so a different approach was needed.

**Attempt 2 — Source code inspection**
- Viewed page source (Ctrl+U) / used browser DevTools on the app's
  front-end JavaScript.
- Found the following block:
```javascript
  var isAdmin = false;
  if (isAdmin) {
      var topLinksTag = document.getElementsByClassName("top-links")[0];
      var adminPanelTag = document.createElement('a');
      adminPanelTag.setAttribute('href', '/admin-bcr1xa');
      adminPanelTag.innerText = 'Admin panel';
      topLinksTag.append(adminPanelTag);
      var pTag = document.createElement('p');
      pTag.innerText = '|';
      topLinksTag.appendChild(pTag);
  }
```
- This script only *renders* the admin panel link into the page if
  `isAdmin` is `true` — a purely client-side, cosmetic check.
- Since my account has `isAdmin = false`, the block never executes,
  and the link never appears in the UI.
- But the admin URL itself — `/admin-bcr1xa` — is sitting right there
  in the JS source, fully readable, regardless of the `isAdmin` value.

## Approach
1. Extracted the hardcoded path `/admin-bcr1xa` from the JS.
2. Navigated directly to that URL in the browser.
3. Admin panel loaded successfully — no server-side check blocked
   the request, even though my session had `isAdmin = false`
   according to the client-side script.
4. Located user `carlos` in the admin panel and deleted the account.

## Vulnerability
- **Unprotected functionality via client-side access control**
  (CWE-284 / broken access control — same root category as Lab 2,
  different disclosure vector).
- The `isAdmin` check exists only in JavaScript running in the
  browser — it decides what gets **displayed**, not what's
  **allowed**. This kind of check is trivially bypassed because:
  - The attacker fully controls the client (browser/JS execution
    environment) — any client-side variable or condition can be
    inspected, ignored, or tampered with.
  - Hardcoding the real admin URL directly in JS, even inside an
    `if` block that "hides" it, still ships that string to every
    user's browser on page load.
- No server-side authorization check existed on the `/admin-bcr1xa`
  route itself — the endpoint trusted that no one would find/request
  the URL.

## Solution (how this should be fixed)
- Never make authorization decisions in client-side JavaScript —
  it can only control UI/display, never enforce actual restriction.
- Enforce authorization **server-side** on every sensitive route:
  check the authenticated user's role/permissions on each request to
  `/admin-*`, independent of how the client got there.
- Avoid embedding sensitive paths, feature flags, or internal
  endpoint names in front-end code shipped to all users — if it's in
  the JS bundle, assume every user (and attacker) can read it.
- Consider returning identical UI for all users and gating purely on
  server response (403/redirect) for unauthorized roles, rather than
  conditionally injecting links.

## Key Takeaway
Same underlying flaw as Lab 2 (unprotected functionality / security
by obscurity), different discovery method: this time the disclosure
came from **client-side JS containing a hardcoded server-trusted
secret path**, rather than `robots.txt`. Confirms a broader pattern:
whenever "protection" = hiding a URL rather than checking permissions
server-side, the URL will eventually leak — via config files,
JS source, browser history, referrer headers, etc. Access control
must always be enforced on the server, never assumed from the
client.

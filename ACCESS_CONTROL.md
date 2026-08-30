# Lab: Unprotected Admin Functionality

**Source:** PortSwigger Web Security Academy — Access Control
**Difficulty:** Apprentice
**Date:** 30 Aug 2026

---

## Objective
Access the unlinked admin panel and delete the user `carlos`.

## Recon / Observations
- Logged in as a normal user, browsed "My account" — no link to any
  admin panel anywhere in the UI.
- Guessing common admin paths (`/admin`, `/administrator`, etc.)
  didn't work directly.
- Checked `/robots.txt` — a good habit for finding paths the app
  doesn't want indexed/linked, but which still exist and are
  reachable:
User-agent: *
Disallow: /administrator-panel

- `robots.txt` disclosed the exact admin path.

## Approach
1. Navigated directly to `/administrator-panel`.
2. Panel loaded with no additional auth/role check — normal user
   session was enough to access admin functionality.
3. Found `carlos` in the user list and deleted the account.

## Vulnerability
- Classic **unprotected functionality** (vertical privilege
  escalation, CWE-284 / broken access control).
- The admin panel had no server-side check confirming the requesting
  user actually has admin privileges — it relied only on the URL
  being unlinked/hidden ("security by obscurity").
- `robots.txt` — meant to guide search engine crawlers — ended up
  leaking the existence of the sensitive path to an attacker.

## Solution (how this should be fixed)
- Enforce server-side authorization checks on every admin route,
  independent of whether the URL is publicly linked.
- Never rely on obscurity (hidden links, unindexed paths) as a
  security control.
- Avoid disclosing sensitive paths in `robots.txt`; if crawling needs
  to be blocked, prefer authentication-gating the route itself over
  just hiding it from crawlers.

## Key Takeaway
Hiding a link from the UI is not access control. Any endpoint —
including ones nobody linked to — needs its own server-side
permission check. `robots.txt` is a useful recon source precisely
because devs sometimes use it to "hide" things instead of properly
restricting access.

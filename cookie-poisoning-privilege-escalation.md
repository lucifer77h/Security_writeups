# Cookie Poisoning: Guest → Admin Privilege Escalation

**Platform:** HackerDNA (CTF practice lab)
**Category:** Session Tampering / Broken Access Control
**Difficulty:** Very Easy

## Objective
Exploit a cookie poisoning vulnerability in a corporate portal to escalate 
privileges from `guest` to `admin` and capture the flag.

## Recon / Observations
- Started with view-source on the page — no useful info in the HTML/comments
- Tried direct path guessing (`/admin`) — no dice
- Checked `robots.txt` for hidden paths — nothing there either
- Recalled a similar client-side trust issue from a PortSwigger lab and 
  pivoted to inspecting how the app tracks session state instead of 
  guessing routes

## Approach
1. Opened DevTools → **Application** tab → **Cookies**
2. Found a session cookie holding what looked like an encoded value
3. Copied the cookie value and switched to the **Console** tab
4. Decoded it client-side:
```js
   atob(cookieValue)
```
5. Output was a JSON object containing session details — including a 
   `role` field set to `"guest"`

## Vulnerability
The application stores the user's role inside a **Base64-encoded, 
client-readable cookie** with no server-side integrity check (no 
signature, no HMAC, no server-side session lookup). Since Base64 is 
encoding, not encryption, the payload is trivially decodable and 
re-encodable by anyone with browser DevTools.

**Root cause:** Trusting client-supplied data for authorization decisions.

## Solution / Exploitation
1. Modified the decoded JSON, changing `"role": "guest"` → `"role": "admin"`
2. Re-encoded it:
```js
   btoa(JSON.stringify(modifiedObject))
```
3. Replaced the cookie value in the **Application** tab with the new 
   encoded string
4. Refreshed the page → now authenticated as `admin` → flag captured

## Fix Recommendation
- Never store authorization state (roles, permissions) in a client-side 
  cookie without server-side validation
- Use signed/encrypted session tokens (e.g., JWT with server-verified 
  signature, or opaque session IDs mapped to server-side session store)
- Re-validate role/permissions server-side on every privileged request — 
  never trust the client's claim of its own role

## Key Takeaway
Encoding is not encryption. Base64 just changes the representation of 
data, not its trustworthiness. Any state that affects authorization 
must be verified server-side — if the client can read it, assume the 
client can rewrite it too.

# Secrets in Source 2: Bypass Security Through Obscurity

| | |
|---|---|
| **Platform** | `<platform name>` |
| **Difficulty** | Very Easy |
| **Category** | Web & API Security |
| **Points** | +50 XP |
| **Solved** | Oct 4, 2026 |
| **Tags** | Client-Side Security, Security Through Obscurity, Browser DevTools, View Source, Information Disclosure, Broken Access Control |

---

## 1. Challenge Description

> SecureVault bet their secrets on security through obscurity, disabling right-click, blocking DevTools, and adding detection scripts. Bypass every client-side block, read the source they tried to hide, and capture the flag.

**Goal:** Read the page source despite the client-side restrictions and find the flag.

---

## 2. TL;DR

The site tries to stop users from viewing its source by disabling right-click and F12 using JavaScript. These controls only exist in the browser, so they can be sidestepped by requesting the source through the `view-source:` URL scheme. The source contained a developer `TODO` comment that leaked the path of a hidden flag page, `/secretvault/flag`, which was accessible with no authentication.

---

## 3. Recon

Opening the target in the browser:

- Right-click is disabled.
- `F12` does nothing.
- Detection scripts try to block DevTools.

So the normal routes to the source are blocked. The restrictions are all enforced by JavaScript running in **my** browser, which is the key weakness.

---

## 4. Exploitation

### Step 1: Bypass the client-side blocks with `view-source:`

Prefix the target URL with `view-source:`:

```
view-source:https://<target-url>/
```

This makes the browser fetch and display the raw HTML as text, **without executing the page's JavaScript**, so the anti-right-click and anti-DevTools scripts never run.

### Step 2: Read the source

Inside the HTML there was a developer comment, roughly:

```html
<!-- TODO: secret vault is in readable form, move it ... -->
```

It also disclosed the path of the flag page:

```
/secretvault/flag
```

### Step 3: Visit the leaked endpoint

```
https://<target-url>/secretvault/flag
```

The page loaded without any authentication and displayed the flag.

### Flag

```
FLAG{...redacted...}
```

---

## 5. Vulnerability Analysis

| Issue | Description |
|---|---|
| **Security through obscurity** | Hiding the source with JS tricks is not a security control. Anything sent to the browser is readable by the user. |
| **Client-side enforcement** | Right-click and F12 blocking runs on the client, which the attacker controls. It can be bypassed, disabled, or never executed at all. |
| **Information disclosure** | A leftover `TODO` comment revealed a sensitive internal path. |
| **Broken access control** | `/secretvault/flag` served sensitive content to anyone who knew the URL. No authentication or authorization check. |

---

## 6. Root Cause

The developers assumed that if users cannot easily open the source, they cannot find the secret. The actual protection of the flag relied entirely on **nobody knowing the URL**, rather than on a server-side access control check.

---

## 7. Remediation

1. **Never rely on obscurity.** Treat everything delivered to the client as public.
2. **Enforce access control on the server.** Sensitive routes like `/secretvault/flag` must require authentication and authorization.
3. **Strip developer comments and TODOs** from production builds (build-step minification or comment removal).
4. **Don't waste effort on anti-DevTools scripts.** They add no real security and only slow down legitimate users.
5. **Review responses for leaked paths, keys, and internal notes** before deploying.

---

## 8. Key Takeaways

- **Client-side controls are not security controls.** If the browser can see it, so can the user.
- **`view-source:` is a URL scheme, not a DevTools feature.** It works even when right-click and F12 are blocked, because it never runs the page's scripts.
- **Always read the source.** Comments, hidden paths, and `TODO`s are classic information disclosure.
- **Watch the URL bar.** I had used view-source many times without ever noticing how the URL changes. In security, small details like that are often the whole solution.

---

## 9. Tools Used

- Web browser (`view-source:` scheme)

---

*Writeup by [lucifer77h](https://github.com/lucifer77h)*

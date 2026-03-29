# SECURITY_REPORT.md

**Application:** https://muthummk.online  
**Date:** 2026-03-28  
**Scope:** API + Web Application  

---

# 1. Vulnerability: Missing SendGrid API Key Configuration

## OWASP Category
A05: Security Misconfiguration

---

## Affected File & Line Number
- `/api/sendgrid` (backend service)
- Line: Environment configuration (process.env.SENDGRID_API_KEY usage)

---

## Severity
HIGH

---

## Description
The application email service is dependent on SendGrid, but the API key is not configured in the production environment. As a result, the service fails at runtime and returns a configuration error instead of handling the failure securely.

---

## Business Impact
- Email/contact system is non-functional
- Loss of user inquiries or notifications
- Production service disruption
- Exposure of internal configuration state to external users

---

## Proof of Concept

```bash id="poc_001"
curl -X POST https://muthummk.online/api/sendgrid
````

### Response:

```json id="resp_001"
{"error":"SendGrid API key not configured"}
```

---

## Recommended Fix

### Secure environment handling:

```js id="fix_001"
if (!process.env.SENDGRID_API_KEY) {
  return res.status(500).json({
    error: "Service temporarily unavailable"
  });
}
```

### Store secrets securely:

* AWS Secrets Manager
* Environment variables in CI/CD pipeline

---

# 2. Vulnerability: Missing API Rate Limiting

## OWASP Category

A04: Insecure Design

---

## Affected File & Line Number

* `/api/sendgrid` endpoint
* Express/Nginx API layer (no rate limiting configured)

---

## Severity

HIGH

---

## Description

The API endpoint does not implement any rate limiting mechanism, allowing unlimited requests from a single IP. This makes the system vulnerable to abuse, spam, and potential denial-of-service attacks.

---

## Business Impact

* Email spam attacks
* Increased infrastructure cost
* API service degradation
* Potential denial-of-service (DoS)
* Abuse of SendGrid quota (if enabled)

---

## Proof of Concept

```bash id="poc_002"
for i in $(seq 1 20); do
  curl -X POST https://muthummk.online/api/sendgrid \
  -H "Content-Type: application/json" \
  -d '{"message":"spam test"}'
done
```

---

## Recommended Fix

### Express rate limiting:

```js id="fix_002"
import rateLimit from "express-rate-limit";

const sendgridLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: "Too many requests. Please try again later."
});

app.use("/api/sendgrid", sendgridLimiter);
```

---

# 3. Vulnerability: Input Validation Weakness (Potential XSS)

## OWASP Category

A03: Injection

---

## Affected File & Line Number

* `/api/sendgrid` request body processing
* Message field input handling

---

## Severity

MEDIUM

---

## Description

The API accepts user input without strict validation or sanitization. Although no execution occurs currently, unsanitized input may lead to XSS or injection vulnerabilities if rendered in frontend or stored in database.

---

## Business Impact

* Stored Cross-Site Scripting (XSS)
* Data integrity issues
* Compromise of user session (if frontend vulnerable)
* Email template injection risk

---

## Proof of Concept

```json id="poc_003"
{
  "name": "test",
  "email": "x@x.com",
  "message": "<script>alert(1)</script>"
}
```

---

## Recommended Fix

### Input sanitization:

```js id="fix_003"
import validator from "validator";

message = validator.escape(message);
```

### Schema validation:

```js id="fix_003b"
import { z } from "zod";

const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  message: z.string().max(500)
});
```

---

# 4. Vulnerability: Information Disclosure via API Error Messages

## OWASP Category

A05: Security Misconfiguration

---

## Affected File & Line Number

* `/api/sendgrid` error handler

---

## Severity

MEDIUM

---

## Description

The API exposes internal configuration state in error messages, revealing that SendGrid is used and whether API keys are missing.

---

## Business Impact

* System fingerprinting by attackers
* Exposure of internal architecture
* Increased attack surface for targeted exploitation

---

## Proof of Concept

```json id="poc_004"
{"error":"SendGrid API key not configured"}
```

---

## Recommended Fix

```js id="fix_004"
return res.status(500).json({
  error: "Request failed. Please try again later."
});
```

---

# 5. Vulnerability: Missing Request Size Limiting

## OWASP Category

A04: Insecure Design

---

## Affected File & Line Number

* Express API middleware

---

## Severity

LOW

---

## Description

The API does not restrict payload size, allowing potential abuse via large request bodies.

---

## Business Impact

* Memory exhaustion risk
* DoS via large payload attacks
* Backend performance degradation

---

## Proof of Concept

```bash id="poc_005"
curl -X POST https://muthummk.online/api/sendgrid \
-H "Content-Type: application/json" \
-d '{"message":"'$(head -c 50000 /dev/zero | tr '\0' 'A')'"}'
```

---

## Recommended Fix

```js id="fix_005"
app.use(express.json({ limit: "10kb" }));
```

---

# 6. Summary

| Severity | Count |
| -------- | ----- |
| HIGH     | 2     |
| MEDIUM   | 2     |
| LOW      | 1     |

---

# 7. Conclusion

The application demonstrates a strong infrastructure security baseline due to proper Nginx hardening and HTTPS enforcement.

However, API-layer security requires improvement in:

* Secret management
* Rate limiting
* Input validation
* Error handling hygiene

Once these issues are resolved, the application will meet **enterprise-grade security standards**.

---

# END OF REPORT

```

---

If you want next upgrade, I can also help you:

✔ convert this into **PDF submission report (interview ready)**  
✔ or format it for **GitHub README + portfolio showcase**  
✔ or add **OWASP scoring badge system (A+ report style)**
```

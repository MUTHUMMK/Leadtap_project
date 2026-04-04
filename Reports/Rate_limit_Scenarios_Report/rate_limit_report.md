# OWASP Security Test Report

## Vulnerability: Rate Limiting Implemented Verification

# Endpoint Tested: https://muthummk.online/
# Method: GET
# Test Type: Rate Limiting Validation
# Severity: Informational (Fix Verified)

![Click ==>> Output_Response_Report_File](<./Output_Response(429_Too_Many_Requests).txt>)
---

## Description
A rate limiting mechanism has been implemented on the API endpoint. This test validates that excessive requests are properly restricted.

---

## Proof of Concept

### Command Used
```bash
for i in {1..80}; do curl -I https://muthummk.online/; done
```

---

## Observed Output

```
HTTP/1.1 429 Too Many Requests
```

---

## Result

- The server successfully blocks excessive requests.
- Rate limiting is functioning as expected.
- The system returns HTTP 429 Too Many Requests after threshold is exceeded.

---

## Security Impact

### Before Fix
- API was vulnerable to abuse
- Possible DoS and spam attacks

### After Fix
- Requests are throttled
- Abuse prevention is active
- Improved system stability

---

## Screenshot Evidence

(Add your terminal screenshot here before submission)

---

## Conclusion

The rate limiting control is correctly implemented and effectively mitigates abuse scenarios.


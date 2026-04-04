### Scenario:
```
I want to flood the business inbox using the contact form and I have a script that can send 1,000
requests per minute.
```
![Click ==>> Real_Time_Check_Report_File ](<./rate_limit_report.md>)

# Vulnerability: Missing Rate Limiting on Contact API
# Vulnerability Name: Missing Rate Limiting on /api/sendgrid
# OWASP Category: A04: Insecure Design
# Affected File & Line Number: /api/sendgrid.js  
# Severity: High

# Description:
```
1.The contact form API endpoint /api/sendgrid does not implement any rate limiting or request throttling.

2.This allows an attacker to send a large number of requests in a short period, leading to abuse of the email service.
```
# Business Impact: (An attacker can)
```
Flood the business inbox with spam emails
Exhaust SendGrid quota (cost impact)
Cause Denial of Service (DoS)
Degrade user experience and trust
```
# Proof of Concept:
Step 1: Attack Script
```
for i in $(seq 1 10); do 
  curl -X POST https://muthummk.online/api/sendgrid \
  -H "Content-Type: application/json" \
  -d '{"name":"test","email":"x@x.com","phone":"0000000000","message":"spam"}'
done
```
Step 2: Observed Result
```
Multiple emails received in inbox 
OR
SendGrid usage increased rapidly 
```
This confirms no rate limiting is applied.

## Recommended Fix:
 
# Rate Limiting via NGINX (Recommended for production)
```
(nginx.conf file)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=5r/m; 
limit_req_status 429;


(app.conf file)
server {
  location /api/sendgrid {
    limit_req zone=api_limit burst=10 nodelay;
    proxy_pass http://localhost:3000;
  }
}
```
# Restart NGINX
```
sudo nginx -t
sudo systemctl reload nginx
```
# Verification After Fix:

Run same attack again:
```
for i in $(seq 1 10); do 
  curl -X POST https://muthummk.online/api/sendgrid \
  -H "Content-Type: application/json" \
  -d '{"name":"test","email":"x@x.com","phone":"0000000000","message":"spam"}'
done
```
# Expected Result After Fix:
```
HTTP/1.1 429 Too Many Requests
```

## Requests are blocked after limit is reached 

### Final Conclusion
```
The vulnerability allows abuse of the email system due to lack of request throttling.
Implementing rate limiting at NGINX and/or application level effectively mitigates this risk.
```

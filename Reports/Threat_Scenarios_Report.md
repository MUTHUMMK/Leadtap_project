# Scenario 1: Inbox Flooding (Spam Attack)
# Attack Goal
```
Attacker wants to flood business email inbox using contact form.
```
# How the Attack Works

If /api/sendgrid has no:
```
rate limiting
captcha
IP blocking
```
Attacker can automate requests using scripts.

# Proof of Concept (your case)
```
for i in {1..80}; do 
  curl -X POST https://muthummk.online/api/sendgrid \
  -H "Content-Type: application/json" \
  -d '{"name":"test","email":"x@x.com","phone":"000","message":"spam"}'
done
```
# Impact
```
Inbox spam flood 
SendGrid quota exhaustion 
Possible service downtime
```
# Fix
```
Rate limiting (NGINX or backend)
CAPTCHA (Google reCAPTCHA)
IP throttling
```
###
# Scenario 2: Email Injection (Malicious Email Content)
# Attack Goal
```
Inject malicious links into email content sent from system.
```
# How the Attack Works

If user input is not sanitized:
```
attacker adds HTML links or scripts
email gets sent as “trusted business email”
```
# Proof of Concept
```
{
  "name": "user",
  "email": "test@test.com",
  "message": "<a href='http://malicious.com'>Click here</a>"
}
```
# Impact
```
Phishing attacks using trusted domain
Business reputation damage
```
# Fix
```
Sanitize input before sending email
Use HTML escaping
Allow only plain text emails
```
###

# Scenario 3: Source Code Exposure
# Attack Goal

Access full application code via browser.

# How the Attack Works

If:
```
source maps exposed
misconfigured server
directory listing enabled
```
Attacker can inspect frontend code.

# Proof of Concept
```
https://muthummk.online/_next/static/chunks/

(or)
view source:
Right click → View Page Source
```
# Impact
```
Business logic exposure
API endpoints revealed
Attack surface increases
```
# Fix
```
Disable source maps in production
Restrict directory listing
Secure build configuration
```
# Scenario 4: Clickjacking Attack
# Attack Goal

Embed your website inside malicious site.

# How the Attack Works

If no frame protection header:
```
<iframe src="https://muthummk.online"></iframe>
```
User thinks it's safe but interacts inside hidden iframe.

# Proof of Concept

Try embedding your site in iframe (external HTML page)

# Impact

```
User deception
Fake clicks / actions
Security bypass
```
# Fix

Add NGINX header:
```
add_header X-Frame-Options "DENY";
(or)
frame-ancestors 'none'
```
# Scenario 5: Root-Level App Risk
# Attack Goal

Understand risk if app runs as root.

# How Attack Works

If Node.js runs as root:
```
any RCE vulnerability = full server takeover
```
# Impact
```
Full system compromise 
SSH key theft
Database destruction
```
# Fix

NEVER run app as root.
```
sudo adduser appuser
sudo chown -R appuser:appuser app
```
Run:
```
pm2 start app --user appuser
```

# OWASP Threat Scenario Analysis

This report analyzes real-world attack scenarios against the deployed Next.js application and demonstrates exploitation patterns with mitigation strategies.
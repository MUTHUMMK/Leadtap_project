## Header vulnerability:

### Vulnerability 1: Missing Security Headers
### Vulnerability Name: Missing Security Headers
### OWASP Category: A05: Security Misconfiguration
### Affected File & Line Number: NGINX / Next.js configuration (headers not set)
### Severity: Medium
### Description:

#### The application response headers are missing important security protections such as:

```
* Content-Security-Policy
* X-Frame-Options
* X-Content-Type-Options
* Strict-Transport-Security
```

### From your response:
![Click ==>> Missing_Header_Screenshot](<./screenshots/Missing_Header_Screenshot.png>)

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
X-Powered-By: Next.js
```
These headers indicate default configuration without security hardening.

### Business Impact: (An attacker can)
```
Perform Clickjacking attacks (no X-Frame-Options)
Execute XSS attacks more easily (no CSP)
Exploit MIME sniffing vulnerabilities
Downgrade HTTPS attacks (no HSTS)
```
### Proof of Concept:
```
curl -I https://muthummk.online
```
Observe missing headers in response.

### Fix:

### Option 1: Fix in NGINX (app.conf)

![Click ==>> Header_added_after_Screenshot](<./screenshots/Header_Added_Screenshot.png>)
```
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self';";
```

### Option 2: Fix in Next.js (next.config.js)
```
module.exports = {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          { key: "X-Frame-Options", value: "DENY" },
          { key: "X-Content-Type-Options", value: "nosniff" },
          { key: "Content-Security-Policy", value: "default-src 'self'" }
        ],
      },
    ];
  },
};

```

### Vulnerability 2: Information Disclosure via Headers
### OWASP Category: A05: Security Misconfiguration
### Severity: Low
### Description:
The server exposes technology stack:
```
Server: nginx/1.18.0 (Ubuntu)
X-Powered-By: Next.js
```
### Impact:
```
Helps attackers identify known vulnerabilities
Makes targeted attacks easier
```
### Proof of Concept:
```
curl -I https://muthummk.online
```
### Fix: NGINX (app.conf)
```
server_tokens off;
```

## Dependency vulnerabilities:

![Click ==>> npm_audit_Report_File](<./Reports/SAST_DAST_Output_Report/npm_audit_report>)

### Vulnerability 1: Vulnerable Next.js Version
### Vulnerability Name: Outdated Next.js Framework with Multiple Critical Vulnerabilities
### OWASP Category: A06: Vulnerable and Outdated Components
### Affected File & Line Number: package.json → next dependency
### Severity: Critical
### Description:

The application is using a vulnerable version of Next.js, which contains multiple critical security issues such as:
```
Remote Code Execution (RCE)
Server-Side Request Forgery (SSRF)
Authorization Bypass
Cache Poisoning
Denial of Service (DoS)
```
These vulnerabilities exist due to outdated dependencies.

### Business Impact: (An attacker can)
```
Execute arbitrary code on the server
Bypass authentication
Crash the application (DoS)
Access internal services (SSRF)
```
### Proof of Concept:
```
npm audit
```

### Output shows:
```
Severity: critical
```
Next.js is vulnerable to RCE in React flight protocol
### Fix:
```
npm audit fix --force
```
OR
```
npm install next@latest
```

### Vulnerability 2: Axios DoS Vulnerability
### Vulnerability Name: Axios Denial of Service (DoS)
### OWASP Category: A06: Vulnerable and Outdated Components
### Affected File: package.json → axios
### Severity: High
### Description:

The application uses a vulnerable version of Axios which does not properly validate data size and allows prototype pollution.

### Business Impact:
```
Application crash via large payload
Memory exhaustion
Service downtime
```
### Proof of Concept:
```
npm audit
```
### Recommended Fix:
```
npm audit fix
```

### Vulnerability 3: form-data Critical Weak Randomness
### Vulnerability Name: Weak Random Boundary Generation
### OWASP Category: A02: Cryptographic Failures
### Affected File: node_modules/form-data
### Severity: Critical
### Description:

The form-data package uses unsafe randomness for boundary generation.

### Business Impact:
```
Predictable request boundaries
Potential request manipulation
```
### Proof of Concept:

```
npm audit
```
### Recommended Fix:
```
npm audit fix
```

### Vulnerability 4: Lodash Prototype Pollution
### Vulnerability Name: Prototype Pollution in Lodash
### OWASP Category: A03: Injection
### Affected File: node_modules/lodash
### Severity: High
### Description:

The Lodash library allows modification of object prototypes.

### Business Impact:
```
Modify application behavior
Bypass validation
Potential code execution
```
### Proof of Concept:

```
npm audit
```
### Fix:
```
npm update lodash
```


### Vulnerability 5: glob Command Injection
### Vulnerability Name: Command Injection via glob CLI
### OWASP Category: A03: Injection
### Severity: High
### Description:

The glob package allows execution of commands via CLI flags.

### Business Impact:
```
Arbitrary command execution
Server compromise
```
### Proof of Concept
```
npm audit
```
### Recommended Fix

```
npm audit fix
```

### Vulnerability 6: ReDoS Vulnerabilities (Multiple Packages)
### Vulnerability Name: Regular Expression Denial of Service (ReDoS)
### OWASP Category: A06: Vulnerable Components
### Affected Packages:
```
brace-expansion
minimatch
picomatch
yaml
```
### Severity: Medium–High
### Description:

These libraries contain inefficient regex patterns causing excessive CPU usage.

### Business Impact:

```
Server slowdown
Application crash
```
### Proof of Concept:
```
npm audit
```
### Fix:
```
npm audit fix
```
### Updated Summary:
```
Severity	Count
Critical	2
High	5
Medium	2
```

## SAST Report:

![Click ==>> SonarScanner Report screenshots ](<./Reports/SAST_DAST_Output_Report/Sonar_report.png>)

## DAST (OWSAP ZAP) Report:

![Click ==>> OWSAP ZAP Output Report File](<./Reports/SAST_DAST_Output_Report/zap-report-0ca296b3.html>)
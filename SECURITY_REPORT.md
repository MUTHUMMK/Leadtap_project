# 🛡️ Application Security Assessment & Infrastructure Hardening Report

## Including OWASP Top 10 Mapping, Threat Scenarios, and Mitigation Strategies

---

## Security Controls Implemented

This application follows a **defense-in-depth strategy** by implementing security controls at the server, application, and dependency levels.

---

## 1. Secure SSH Configuration (Server Hardening):

### OWASP Category

* A05: Security Misconfiguration
* A07: Identification & Authentication Failures

### Implementation

* Disabled root login (`PermitRootLogin no`)
* Configured custom SSH port (e.g., `222`)
* Enforced key-based authentication
* Integrated Fail2Ban to prevent brute-force attacks

### Behavior

* Unauthorized login attempts → blocked
* Only authorized users with SSH keys can access

### Security Benefit

* Prevents brute-force and unauthorized root access
* Reduces attack surface by hiding default SSH port

---

## 2. Fail2Ban (Intrusion Prevention):

### OWASP Category

* A09: Security Logging & Monitoring Failures
* A04: Insecure Design

### Implementation

* Monitors:

  * `/var/log/auth.log` (SSH attacks)
  * `/var/log/nginx/access.log` (API abuse)
* Automatically bans malicious IPs via firewall

### Behavior

* Multiple failed attempts → IP banned
* Supports configurable retry and ban duration

### Security Benefit

* Real-time attack detection and blocking
* Prevents repeated abuse from same attacker

---

## 3. NGINX Security Hardening:

### OWASP Category

* A05: Security Misconfiguration

### Implementation 

### Security Headers Configured

```
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self';";
```

### Behavior & Protection

| Header                 | Protection                 |
| ---------------------- | -------------------------- |
| X-Frame-Options        | Prevents clickjacking      |
| X-Content-Type-Options | Stops MIME sniffing        |
| X-XSS-Protection       | Basic XSS protection       |
| HSTS                   | Forces HTTPS               |
| CSP                    | Restricts resource loading |

### Security Benefit

* Protects against XSS, clickjacking, and content injection
* Enforces secure communication (HTTPS only)

---

## 4. NGINX Rate Limiting (DoS Protection):

### OWASP Category

* A04: Insecure Design
* A05: Security Misconfiguration

### Implementation

* Limited requests per IP for `/api` and web routes
* Returns `HTTP 429 Too Many Requests` on abuse

### Behavior

* Normal users unaffected
* Attackers slowed down or blocked

### Security Benefit

* Prevents API abuse and request flooding
* Reduces impact of DoS attacks

---

## 5. Dependency Vulnerability Management

### OWASP Category

* A06: Vulnerable and Outdated Components

### Implementation

* Performed security scan using:

  ```
  npm audit
  ```
* Identified vulnerabilities in dependencies (e.g., axios, lodash, next)
* Applied fixes using:

  ```
  npm audit fix
  npm audit fix --force
  ```

### Behavior

* Vulnerable packages updated to secure versions

### Security Benefit

* Prevents exploitation of known vulnerabilities
* Ensures secure and updated dependencies

---

## Defense-in-Depth Architecture

```
Client → NGINX (Rate Limit + Headers)
       → Application (Next.js)
       → Logs → Fail2Ban → Firewall (IP Ban)
       → SSH Protection (Fail2Ban + Key Auth)
```

---

## Security Outcome

| Layer       | Protection                  |
| ----------- | --------------------------- |
| SSH         | Prevents brute-force login  |
| NGINX       | Rate limiting + headers     |
| Fail2Ban    | Automatic attacker blocking |
| Application | Secure dependencies         |

---

## Summary

The system is secured using a **multi-layered approach aligned with OWASP Top 10**, including:

* Protection against **brute-force attacks**
* Prevention of **DoS and API abuse**
* Mitigation of **web-based attacks (XSS, clickjacking)**
* Continuous monitoring and **automatic IP blocking**
* Secure dependency management

This significantly reduces the overall attack surface and enhances system resilience.

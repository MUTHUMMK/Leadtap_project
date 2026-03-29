# Rate Limiting Implementation Report (Nginx)

##  Scenario

A potential attack scenario was identified where an attacker attempts to flood the business inbox using a contact form.

- Attack capability: Script sending **1000 requests per minute (~17 requests/sec)**
- Goal: Prevent abuse and protect backend services from overload

---

##  Solution: Nginx Rate Limiting

Rate limiting was implemented using Nginx to restrict the number of requests per client IP.

### Configuration

#### 1. Define Rate Limit Zone (nginx.conf)

```nginx
http {
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req_status 429;

    include /etc/nginx/sites-enabled/*;
}
```

#### 2. Apply Rate Limit (Application Config)

```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;

    proxy_pass http://localhost:3000;
}
```

---

##  Configuration Explanation

| Parameter | Description |
|----------|-------------|
| rate=10r/s | Allows 10 requests per second per IP |
| burst=20 | Allows temporary spike of 20 extra requests |
| nodelay | Rejects excess requests immediately |
| limit_req_status 429 | Returns HTTP 429 instead of default 503 |

---

##  Testing Procedure

Simulated attack using curl:

```bash
for i in {1..80}; do curl -I https://yourdomain.com/api; done
```

---

##  Test Result

- Initial requests: Allowed (within rate limit)
- Excess requests: Blocked

### Sample Response

```http
HTTP/1.1 429 Too Many Requests
```

---

##  Analysis

- Attack traffic: ~17 requests/sec
- Configured limit: 10 requests/sec

Result:
- Requests exceeding threshold were successfully blocked
- Server remained stable
- Backend (contact form/email service) protected

---

##  Security Benefits

- Prevents spam attacks on contact forms
- Protects against brute-force attempts
- Mitigates basic DDoS attacks
- Reduces load on backend services

---

##  Considerations

- Very strict limits may affect legitimate users
- Burst value should be tuned based on real traffic
- Rate limiting should be applied mainly to API endpoints

---

##  Conclusion

Nginx rate limiting was successfully implemented and validated. The system effectively blocks excessive requests and returns proper HTTP 429 responses, ensuring application availability and security.

---



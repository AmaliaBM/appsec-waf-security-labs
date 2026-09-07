# Finding — Insufficient Rate Limiting / Scraping Exposure

**Category:** Automated Abuse / Resource Consumption
**Risk:** Medium
**Status:** Laboratory finding
**Environment:** Synthetic

---

# 1. Executive Summary

The application was observed accepting a sustained high request rate from a single source without apparent rate limiting.

Synthetic testing demonstrated:

```text
25 requests/second
10-second test
250 total requests

HTTP 200: 250
HTTP 403: 0
HTTP 429: 0
```

This behavior may facilitate:

* Automated scraping
* Content enumeration
* Authentication endpoint abuse
* Excessive resource consumption
* Automated reconnaissance

---

# 2. Affected Endpoint

Synthetic endpoint:

```text
https://app.example.test/api/products
```

---

# 3. Baseline

Normal synthetic traffic:

```text
10 requests
10 × HTTP 200
```

No errors observed.

---

# 4. Controlled Validation

Authorized laboratory test:

```text
Source IP:
198.51.100.25

Rate:
25 requests/second

Duration:
10 seconds

Total:
250 requests
```

Observed:

```text
HTTP 200: 250
HTTP 403: 0
HTTP 429: 0
```

---

# 5. Apache Reference Configuration

A synthetic `mod_evasive` configuration:

```apache
<IfModule mod_evasive20.c>

    DOSHashTableSize    3097

    DOSPageCount        20
    DOSPageInterval     1

    DOSSiteCount        100
    DOSSiteInterval     1

    DOSBlockingPeriod   600

</IfModule>
```

The conceptual behavior is:

```text
>20 requests/second to a page
        OR
>100 requests/second to the site
        |
        v
temporary blocking
```

The exact semantics depend on the module and its version/configuration.

---

# 6. Application-Level Mitigation

Application-level rate limiting should be implemented using an appropriate distributed mechanism.

Conceptual policy:

```text
Client identity
      |
      v
Rate limiter
      |
      +── within limit → allow
      |
      └── above limit → reject
```

A distributed application should not rely solely on process-local memory.

Possible shared mechanisms include:

```text
Redis
DynamoDB
API Gateway throttling
Dedicated rate-limiting infrastructure
```

---

# 7. AWS WAF Mitigation

AWS WAF Rate-based rules can aggregate requests by source IP.

Synthetic global rule:

```text
Rule name:
    LAB-Global-IP-RateLimit

Aggregation:
    Source IP

Rate limit:
    6000 requests

Evaluation window:
    60 seconds

Action:
    BLOCK
```

Approximation:

```text
6000 / 60
≈ 100 requests/second
```

This provides an approximation of a 100 requests/second per-IP policy.

---

# 8. Endpoint-Specific Rule

Authentication and expensive endpoints may require stricter limits.

Synthetic rule:

```text
Rule name:
    LAB-Connexion-RateLimit

Scope-down:
    URI path = /connexion/

Aggregation:
    Source IP

Rate limit:
    1200 requests

Evaluation window:
    60 seconds

Action:
    BLOCK
```

Approximation:

```text
1200 / 60
≈ 20 requests/second
```

---

# 9. Why Use Multiple Rules?

A layered policy can look like:

```text
                 AWS WAF
                    |
          +---------+---------+
          |                   |
          v                   v
   Global rate limit    Sensitive endpoint
   6000 / 60 sec/IP     1200 / 60 sec/IP
          |                   |
          v                   v
   Whole application       /connexion/
```

This allows sensitive endpoints to have stricter limits without unnecessarily restricting normal traffic throughout the application.

---

# 10. AWS WAF Limitations

The AWS WAF configuration should not be documented as an exact one-second implementation of:

```text
100 requests / 1 second
```

AWS WAF rate-based rules use supported evaluation windows rather than an exact one-second counter.

Similarly, a WAF rate-based rule should not be described as an exact equivalent of:

```text
DOSBlockingPeriod 600
```

from `mod_evasive`.

---

# 11. Scraping Considerations

Per-IP rate limiting can be bypassed by distributed clients.

Example:

```text
IP A → 15 req/s
IP B → 15 req/s
IP C → 15 req/s
IP D → 15 req/s
...
```

Therefore, scraping assessments should also consider:

* Bot detection
* Request fingerprints
* Session behavior
* Authentication state
* User-Agent behavior
* Header consistency
* Endpoint access patterns

Rate limiting is one layer of the control strategy.

---

# 12. Retesting

After implementing the WAF rule, repeat the controlled test.

Synthetic expected behavior:

```text
Test:
25 requests/second

Before:
250 × HTTP 200

After:
Rate-based rule triggered
Some requests rejected
WAF logs show matching rule
```

The exact number of requests blocked should not be treated as deterministic because rate-based enforcement is not an exact per-second counter.

---

# 13. Auditor Conclusion

The application should implement rate limiting at the appropriate application or infrastructure layer.

AWS WAF provides an effective additional control for limiting high-volume traffic, particularly when combined with endpoint-specific rules.

The recommended architecture is:

```text
Application controls
        +
Apache / infrastructure controls
        +
AWS WAF rate limiting
        +
Bot detection
        +
Monitoring
```

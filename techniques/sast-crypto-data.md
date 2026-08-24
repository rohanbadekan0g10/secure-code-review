---
name: sast-crypto-data
description: "SAST categories 4+5: cryptography vulnerabilities (weak algorithms, hardcoded keys/IVs, insecure random, missing cert validation) and sensitive data exposure (secrets in source, verbose errors, sensitive logging, PII in comments)."
---

# Category 4 — Cryptography

## 4.1 Weak Algorithms

| Dangerous | Why |
|---|---|
| DES, 3DES, RC4, Blowfish | Broken or deprecated |
| AES-ECB mode | Deterministic, pattern-preserving |
| RSA < 2048 bits | Factorable |
| SHA1 for signatures | Collision attacks |
| MD5 for anything security-related | Broken |

## 4.2 Hardcoded Keys and IVs

- Encryption keys as string literals in source
- Static IVs/nonces (IV reuse = catastrophic for AES-GCM/CTR)
- Key derivation from predictable seeds (timestamp, PID, hostname)

## 4.3 Insecure Random

- `Math.random()` (JS), `random.random()` (Python), `rand()` (PHP/Ruby) for cryptographic purposes
- `java.util.Random` instead of `java.security.SecureRandom`

## 4.4 Missing Certificate Validation

- `rejectUnauthorized: false` (Node.js)
- `verify=False` (Python requests)
- `InsecureSkipVerify: true` (Go)
- Custom `TrustManager` accepting all certs (Java)
- `CURLOPT_SSL_VERIFYPEER => false` (PHP)

---

# Category 5 — Sensitive Data Exposure

## 5.1 Secrets in Source

Beyond cat 2.1, look for:
- API keys for third-party services (Stripe `sk_live_*`, AWS `AKIA*`, GCP, Azure, Twilio, SendGrid)
- Database connection strings with embedded passwords
- OAuth client secrets / webhook signing secrets
- Encryption keys in non-config files

## 5.2 Verbose Error Messages

- Stack traces returned to client in production (`err.stack` in error handler response)
- Database error messages exposed (table names, column names, query structure)
- `DEBUG = True` in production config
- `display_errors = On` (PHP)

## 5.3 Sensitive Data in Logs

- `console.log(req.body)` on auth/payment endpoints
- `logger.info(f"Login: {username}:{password}")`
- PII (email, SSN, CC number) logged without masking
- Session tokens or API keys written to log files

## 5.4 Sensitive Data in Comments

- TODO/FIXME referencing credentials, internal URLs, or security workarounds
- Commented-out code containing secrets or backdoors
- Production hostnames, internal IP addresses in developer notes

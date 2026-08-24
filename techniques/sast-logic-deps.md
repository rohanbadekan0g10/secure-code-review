---
name: sast-logic-deps
description: "SAST categories 8+9: business logic vulnerabilities (race conditions/TOCTOU, mass assignment, type coercion, integer overflow, missing rate limiting) and dependency analysis (known CVEs, deprecated APIs, unsafe native bindings)."
---

# Category 8 — Business Logic

## 8.1 Race Conditions (TOCTOU)

- Check-then-act without locking: `if (balance >= amount) { deduct(amount) }` without transaction
- File operations: `if (file.exists()) { file.read() }` — file could change between check and read
- DB read-then-write without transaction or row-level lock
- Inventory/balance checks not atomic (concurrent requests can double-spend)

## 8.2 Mass Assignment

- `User.create(req.body)` / `User.update(req.body)` without field allowlist
- `@ModelAttribute` in Spring without `@InitBinder` field restrictions
- `fill()` in Laravel without `$fillable` / with `$guarded` empty
- `attr_accessible` / `strong_parameters` missing in Rails

## 8.3 Unsafe Type Coercion

- `==` instead of `===` in security checks (JS): `"0" == false`, `null == undefined`
- PHP loose comparison: `$password == $hash` (use `hash_equals()`)
- Numeric string comparison in password reset tokens

## 8.4 Integer Overflow

- Arithmetic on user-supplied values without bounds checking
- Price/quantity calculations that could overflow or go negative
- `parseInt()` / `int()` on unbounded input used in array allocation or payment math

## 8.5 Missing Rate Limiting

- Login endpoints without rate limiting middleware
- Password reset without throttling
- OTP/2FA verification without attempt limiting
- API endpoints with no limit on expensive operations (LLM calls, file processing)

---

# Category 9 — Dependency Analysis

## 9.1 Known Vulnerable Packages

Read dependency files and check versions:
- `package.json` / `package-lock.json`
- `requirements.txt` / `Pipfile.lock` / `poetry.lock`
- `pom.xml` / `build.gradle`
- `Gemfile.lock` / `composer.lock` / `go.sum` / `Cargo.lock`

Flag:
- Unpinned deps (`*` `latest` `>=`)
- Packages with CVEs in training data
- Deps not updated in 2+ years vs known EOL

Note: confidence:MEDIUM — training data may not include latest advisories. Recommend `npm audit` / `pip-audit` for authoritative results.

## 9.2 Deprecated API Usage

- Deprecated stdlib functions: Node `Buffer()` without `new`, Python `cgi` module (3.13+)
- Deprecated framework methods with security implications
- EOL runtime versions in `.node-version`, `.python-version`, `Dockerfile FROM`

## 9.3 Unsafe Native Bindings

- Native C/C++ addons without memory safety review → flag for manual audit
- Python `ctypes` / `cffi` without input validation on size/type params
- `unsafe` blocks in Rust → flag for manual review
- CGO calls in Go with user-controlled input

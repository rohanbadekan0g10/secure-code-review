---
name: output-dev
description: "Developer-friendly output format for secure-code-review — ❌ NEVER / ✅ ALWAYS style with fix suggestions, test-to-write, and pre-deployment checklist. Default when any extension flag is set (--dev, --audit, --backdoor, --devops, --diff, --emit-ci) and --report is NOT passed."
---

# Developer-Friendly Output Format

## Per-Finding Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[SEVERITY] Finding Class  ·  path/to/file.ext:line
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ❌ NEVER do this:
     <exact vulnerable code snippet from the file>

  ✅ ALWAYS do this:
     <corrected code snippet — specific to this language/framework>

  Why: <one sentence explaining the real-world impact>

  Test to write:
     <specific test assertion that would catch this regression>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Example

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[CRITICAL] SQL Injection  ·  src/services/userService.js:47
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ❌ NEVER do this:
     db.query(`SELECT * FROM users WHERE name = '${data.name}'`)

  ✅ ALWAYS do this:
     db.query('SELECT * FROM users WHERE name = $1', [data.name])

  Why: String interpolation in SQL lets an attacker inject arbitrary SQL
       commands, accessing or deleting any data in the database.

  Test to write:
     Pass name = "'; DROP TABLE users; --" and assert the query
     executes safely (returns 0 rows, does not throw or modify data).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Summary Line (after all findings)

```
Found: N CRITICAL  N HIGH  N MEDIUM  N LOW
```

## --audit Verdict Block (before checklist, if --audit was passed)

```
═══════════════════════════════════════════════════════
  THIRD-PARTY AUDIT VERDICT: [SAFE TO USE / REVIEW BEFORE USING / DO NOT USE]
═══════════════════════════════════════════════════════
  Scanned: NNN files  |  NNN lines  |  N modules

  [SEVERITY] file:line — description
  ...

  Action: <specific next step>
═══════════════════════════════════════════════════════
```

Verdicts:
- **SAFE TO USE** — zero findings after false-positive filter
- **REVIEW BEFORE USING** — HIGH or MEDIUM findings; inspect before running
- **DO NOT USE** — CRITICAL finding confirmed; do not install or execute

## Pre-Deployment Checklist (always append at end)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PRE-DEPLOYMENT SECURITY CHECKLIST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Secrets & Config
  [ ] No hardcoded secrets, keys, or passwords in source
  [ ] All secrets loaded from environment variables
  [ ] .env is in .gitignore and never committed

Authentication
  [ ] Passwords hashed with bcrypt / argon2 / scrypt (NOT md5/sha1)
  [ ] Session tokens: httpOnly + Secure + SameSite=Strict
  [ ] No sensitive data in localStorage or sessionStorage

Input & Data
  [ ] All user input validated server-side before use
  [ ] Parameterized queries everywhere (no string SQL concatenation)
  [ ] File uploads: type, size, and extension validated

Infrastructure (if IaC detected)
  [ ] No public cloud storage buckets or overly open security groups
  [ ] Containers run as non-root user
  [ ] Base images pinned to specific versions (not :latest)

Dependencies
  [ ] No known-vulnerable package versions (npm audit / pip-audit)
  [ ] Dependency versions pinned in lock file and committed

CI/CD
  [ ] GitHub Actions: all third-party actions SHA-pinned
  [ ] Secrets not echoed in pipeline logs

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## --emit-ci Output (if --emit-ci was passed)

After checklist, read and write CI templates:

1. Read `references/ci-templates/github-actions.yml`
2. Write to `<scanned-path>/.github/workflows/security.yml` (create dirs if needed)
3. Read `references/ci-templates/pre-commit-config.yaml`
4. Write to `<scanned-path>/.pre-commit-config.yaml`
5. Print:
```
CI configs generated:
  ✓ .github/workflows/security.yml  — GitHub Actions (Gitleaks + Semgrep + Trivy + Checkov)
  ✓ .pre-commit-config.yaml         — pre-commit hooks

Setup: pip install pre-commit && pre-commit install
Note: These tools are optional. /secure-code-review works without them.
```

# secure-code-review

> **Universal security code review for engineering teams** — works in Claude Code, GitHub Copilot, Visual Studio, VS Code, and any IDE. No external tools, no installs, no API keys.

[![Works with Claude Code](https://img.shields.io/badge/Claude%20Code-skill-blue)](https://claude.ai/code)
[![Works with GitHub Copilot](https://img.shields.io/badge/GitHub%20Copilot-compatible-green)](https://github.com/features/copilot)
[![VS Code](https://img.shields.io/badge/VS%20Code-tasks%20included-blue)](https://code.visualstudio.com)
[![Visual Studio](https://img.shields.io/badge/Visual%20Studio-Copilot%20compatible-purple)](https://visualstudio.microsoft.com)

---

## What It Does

`/secure-code-review` is a slash skill for [Claude Code](https://claude.ai/code) that performs deep security analysis on your codebase. Claude reads your source files directly, traces data flows from user input to dangerous sinks, and reports exploitable vulnerabilities — not just pattern matches.

- **10 vulnerability categories** — injection, auth, authorization, crypto, data exposure, input validation, configuration, business logic, dependencies, and language-specific issues
- **Universal language support** — auto-detects the language from signal files; no configuration needed
- **Developer-friendly output** — ❌ NEVER / ✅ ALWAYS format with fix suggestions and tests to write
- **Third-party audit** — scan open-source or vendor code for backdoors before adopting it
- **IaC security** — Dockerfile, Terraform, Kubernetes, GitHub Actions (auto-triggered)
- **CI integration** — optionally generates GitHub Actions and pre-commit configs

---

## Language Support

| Tier | Languages | Detection |
|---|---|---|
| **Tier 1** (full coverage) | JavaScript, TypeScript, Python, Java, PHP, C#, Go, Ruby | `package.json`, `pyproject.toml`, `pom.xml`, `composer.json`, `*.csproj`, `go.mod`, `Gemfile` |
| **Tier 2** (core coverage) | Rust, Kotlin, Swift, Scala | `Cargo.toml`, `build.gradle.kts`, `Package.swift`, `build.sbt` |
| **Tier 3** (generic) | Any other language | Pattern-based analysis |

---

## Installation

Copy the skill to your Claude Code skills directory:

```bash
# macOS / Linux
git clone https://github.com/<your-org>/secure-code-review \
  ~/.claude/skills/secure-code-review

# Windows (PowerShell)
git clone https://github.com/<your-org>/secure-code-review `
  "$env:USERPROFILE\.claude\skills\secure-code-review"
```

Restart Claude Code — the skill is immediately available as `/secure-code-review`.

> **No further setup required.** Claude Code is the analysis engine. No `npm install`, no `pip install`, no Docker.

---

## Usage

```
/secure-code-review <path> [flags]
```

### Standard scan

```bash
/secure-code-review ~/code/my-app
```

Scans all source files, auto-detects language and framework, traces data flows, and outputs findings in developer-friendly format with fix suggestions.

### All flags

| Flag | Description |
|---|---|
| `--quick` | Scan only auth, config, and entry-point files (fastest) |
| `--thorough` | Full call-chain tracing across all files (most complete) |
| `--dev` | Force developer-friendly ❌/✅ output |
| `--report` | Formal `SF<NN>` output for security teams (includes `sast-findings/` directory) |
| `--audit` | Third-party code evaluation — outputs SAFE / REVIEW / DO NOT USE verdict |
| `--backdoor` | Backdoor and malware scan (6 detection types + entropy analysis) |
| `--devops` | IaC scan (auto-triggered when Dockerfile/Terraform/K8s/CI files detected) |
| `--diff` | Scan only files changed in the current branch (PR review mode) |
| `--emit-ci` | Generate GitHub Actions and pre-commit security configs |
| `--custom-rules <file>` | Append org-specific rules from a markdown file |
| `--language <lang>` | Force language detection (e.g. `--language python`) |

### Examples

```bash
# Review your own code before merging
/secure-code-review ~/code/my-app

# Quick pre-commit check on changed files only
/secure-code-review ~/code/my-app --diff --quick

# Evaluate a third-party library before adding it as a dependency
/secure-code-review ~/vendor/suspect-lib --audit

# Audit open-source code for backdoors
/secure-code-review ~/downloads/npm-package --backdoor

# Full security review with CI configs generated
/secure-code-review ~/code/my-app --thorough --emit-ci

# Scan Terraform + Kubernetes infra
/secure-code-review ~/infra --devops

# Formal report for security team review
/secure-code-review ~/code/my-app --report
```

---

## Output

### Developer-friendly (default)

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
Found: 2 CRITICAL  3 HIGH  4 MEDIUM  1 LOW
```

### Third-party audit (`--audit`)

```
═══════════════════════════════════════════════════════
  THIRD-PARTY AUDIT VERDICT: DO NOT USE
═══════════════════════════════════════════════════════
  Scanned: 247 files  |  8,340 lines  |  3 modules

  [CRITICAL] postinstall.js:14 — outbound HTTP during npm install
  [HIGH]     src/utils/crypto.js:89 — base64-encoded eval payload
  [MEDIUM]   package.json — event-stream@3.3.6 (known compromised)

  Action: Do not install or execute. Inspect flagged files and
          consider an alternative package.
═══════════════════════════════════════════════════════
```

---

## What It Detects

### Security vulnerabilities (10 categories)

| # | Category | Examples |
|---|---|---|
| 1 | Injection | SQLi, XSS, command injection, SSTI, LDAP, NoSQL, CRLF |
| 2 | Authentication & session | Hardcoded creds, weak hashing, session fixation, JWT flaws |
| 3 | Authorization | IDOR, missing access control, privilege escalation |
| 4 | Cryptography | Weak algorithms, hardcoded keys/IVs, missing cert validation |
| 5 | Data exposure | Secrets in source, verbose errors, sensitive logging |
| 6 | Input validation | Path traversal, file upload bypass, ReDoS, unsafe deserialization |
| 7 | Configuration | CORS/CSP misconfig, insecure cookies, open redirects, debug endpoints |
| 8 | Business logic | Race conditions, mass assignment, type coercion, missing rate limits |
| 9 | Dependencies | Known CVEs, unpinned versions, deprecated APIs |
| 10 | Language-specific | Prototype pollution (JS), JNDI (Java), type juggling (PHP), ERB injection (Ruby), and more |

### Backdoor & malware detection (`--backdoor` / `--audit`)

| Type | What Claude looks for |
|---|---|
| Obfuscated execution | `eval`, `base64_decode`, `fromCharCode`, dynamic imports, hex strings used as code |
| Hidden network callbacks | Hardcoded IPs/domains, unexpected `fetch`/HTTP in utility code, DNS in lifecycle scripts |
| Rogue credentials | Backdoor accounts, hardcoded API keys, secondary auth paths |
| Supply chain tampering | Malicious `postinstall`/`preinstall`, known-compromised package versions |
| CI/CD poisoning | Unpinned GitHub Actions, `pull_request_target` with write access, secrets in logs |
| Timing bombs | Date-gated logic, kill-switch APIs, hidden environment flags |
| High-entropy strings | Shannon entropy > 4.5 on strings > 20 chars = possible embedded payload |

### IaC security (`--devops`, auto on Dockerfile/Terraform/K8s/CI files)

| Platform | Key checks |
|---|---|
| Dockerfile | Root user, remote `ADD`, secrets in `ARG`/`ENV`, unpinned base images |
| Terraform | Public S3, open security groups, unencrypted storage, `Action: "*"` IAM |
| Kubernetes | Privileged containers, host namespaces, secrets in ConfigMap, mutable tags |
| GitHub Actions | Unpinned actions (SHA required), `pull_request_target`, expression injection |

---

## How It Works

This skill uses a **tiered token-optimized architecture**. The router (`SKILL.md`) loads in ~800 tokens and pulls in only the modules needed for each scan.

```
SKILL.md (router, ~80 lines)           ← always loaded
techniques/
  sast-injection.md                    ← loaded for all scans
  sast-auth-authz.md                   ← loaded for all scans
  sast-crypto-data.md                  ← standard + thorough
  sast-input-config.md                 ← standard + thorough
  sast-logic-deps.md                   ← thorough only
  sast-lang-specific.md               ← when Tier 1 language detected
  backdoor-detection.md               ← --backdoor / --audit
  devops-iac.md                        ← auto on IaC files / --devops
  supply-chain.md                      ← --audit
  dataflow.md                          ← tracing methodology
  output-dev.md                        ← developer output format
references/
  route-mapping.md                     ← --thorough / DAST correlation
  output-report.md                     ← --report
  ci-templates/
    github-actions.yml                 ← written by --emit-ci
    pre-commit-config.yaml             ← written by --emit-ci
```

**Subagent orchestration:** When multiple modules apply, they run as parallel fork subagents — wall-clock equals the slowest single module, not the sum of all. A two-stage false-positive filter agent always runs last to remove noise and assign confidence levels.

---

## CI Integration (`--emit-ci`)

Running `--emit-ci` writes two ready-to-use configs to your project:

**`.github/workflows/security.yml`** — GitHub Actions security pipeline (SHA-pinned):
- [Gitleaks](https://github.com/gitleaks/gitleaks) — secrets in git history
- [Semgrep](https://github.com/semgrep/semgrep) — SAST with OWASP Top 10 rules
- [Trivy](https://github.com/aquasecurity/trivy) — dependency CVEs + IaC misconfigs
- [Checkov](https://github.com/bridgecrewio/checkov) — IaC policy enforcement

**`.pre-commit-config.yaml`** — pre-commit hooks:
- Gitleaks — catches secrets before commit
- detect-private-key — blocks accidentally committed keys
- Hadolint — Dockerfile linting

> These tools run in CI/CD. The `/secure-code-review` skill itself never calls them — Claude is the analysis engine.

---

## Extending with Custom Rules

Create a markdown file with org-specific rules and pass it with `--custom-rules`:

```markdown
# My Org Security Rules

## Rule: No MD5 for any purpose
Any use of MD5 (even non-security) is banned by org policy. Flag all occurrences.

## Rule: Internal API calls must use mTLS
Any HTTP call to *.internal.example.com must use the mTLS client from pkg/mtls.
```

```bash
/secure-code-review ~/code/my-app --custom-rules ./security-rules.md
```

---

## Pre-Deployment Checklist

Every review appends a checklist covering secrets, authentication, input validation, infrastructure, dependencies, and CI/CD — a final gate before any code ships.

---

## Universal Setup — Works in Every IDE

### Claude Code (CLI · Desktop · VS Code Extension · Web)

The full skill with all flags. Best experience — deep data-flow tracing, parallel subagents, backdoor detection.

```bash
# Install
git clone https://github.com/YOUR-ORG/secure-code-review \
  ~/.claude/skills/secure-code-review   # macOS/Linux

git clone https://github.com/YOUR-ORG/secure-code-review `
  "$env:USERPROFILE\.claude\skills\secure-code-review"   # Windows

# Use
/secure-code-review ~/code/my-app
/secure-code-review ~/code/my-app --audit
```

Restart Claude Code — `/secure-code-review` appears immediately.

---

### GitHub Copilot (VS Code · Visual Studio · JetBrains · github.com)

Two files to copy into each project repo:

**1. Security awareness on every Copilot suggestion** — copy to `.github/copilot-instructions.md`:

```bash
cp templates/copilot-instructions.md .github/copilot-instructions.md
```

Once added, Copilot will automatically:
- Flag injection sinks as you type
- Suggest parameterized queries instead of string SQL
- Warn on hardcoded secrets and weak crypto
- Enforce `HttpOnly`/`Secure` on cookie suggestions

**2. Full security review on demand** — copy to `.github/prompts/security-review.prompt.md`:

```bash
mkdir -p .github/prompts
cp templates/security-review.prompt.md .github/prompts/security-review.prompt.md
```

Then in Copilot Chat (VS Code or Visual Studio), type:

```
#security-review
```

Or reference specific files:

```
#security-review review #file:src/controllers/userController.js
#security-review check this for SQL injection #selection
```

Copilot will run all 10 vulnerability categories with ❌/✅ fix suggestions.

---

### VS Code Tasks (Claude Code Extension)

If your team uses the Claude Code VS Code extension, the `.vscode/tasks.json` included in this repo adds four commands to the VS Code Task Runner:

| Task | Keyboard |
|---|---|
| Security Review — Full | `Ctrl+Shift+P` → Tasks: Run Task → Security Review — Full |
| Security Review — Quick | Tasks: Run Task → Security Review — Quick |
| Security Review — Changed Files Only | Tasks: Run Task → Security Review — Changed Files Only |
| Security Review — Audit Third-Party Code | Tasks: Run Task → Security Review — Audit Third-Party Code |

Copy `.vscode/tasks.json` to your project repo to enable these tasks.

---

### Org-Wide Rollout Checklist

To deploy across your organization:

```
For every project repo:
  [ ] Copy templates/copilot-instructions.md → .github/copilot-instructions.md
  [ ] Copy templates/security-review.prompt.md → .github/prompts/security-review.prompt.md
  [ ] Copy .vscode/tasks.json → .vscode/tasks.json  (if using VS Code)

For developers who want deep analysis:
  [ ] Install Claude Code: https://claude.ai/code
  [ ] Clone this repo to ~/.claude/skills/secure-code-review
  [ ] Done — /secure-code-review is available immediately
```

---

## Requirements

| Tool | Requirement |
|---|---|
| Claude Code | [Claude Code](https://claude.ai/code) + Claude account (free or Pro) |
| GitHub Copilot | GitHub Copilot subscription + VS Code, Visual Studio, or JetBrains |
| VS Code Tasks | Claude Code VS Code extension |

---

## Architecture Decisions

| Decision | Choice | Reason |
|---|---|---|
| External tools | None required | Works on any machine without setup |
| Language detection | Signal-file based | No config needed; covers monorepos |
| Output format | Developer-friendly default | Engineers fix issues faster with ❌/✅ + tests |
| Token optimization | Tiered modules | 91% token reduction vs monolithic file |
| False positives | Two-stage LLM filter | Reduces noise before results reach the developer |

---

## License

MIT — see [LICENSE](./LICENSE)

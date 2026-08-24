---
name: sast-input-config
description: "SAST categories 6+7: input validation vulnerabilities (file upload, path traversal, ReDoS, unsafe deserialization) and configuration security (CORS, CSP, insecure cookies, debug endpoints, open redirects)."
---

# Category 6 — Input Validation

## 6.1 File Upload Vulnerabilities

- Missing file type validation (accepts any extension)
- MIME type check only (bypassable with magic bytes)
- No file size limit
- Uploaded files stored in web-accessible directory with original filename
- Path traversal in filename: `../../etc/passwd`
- Uploaded files executed (PHP/JSP in upload directory)

## 6.2 Path Traversal

- `path.join(basePath, userInput)` without canonicalization + prefix check
- `os.path.join(base, user_input)` — does NOT prevent `../` in Python
- `File.new(params[:path])` without sanitization
- Missing `realpath()` comparison against allowed base directory

## 6.3 ReDoS (Regex Denial of Service)

Catastrophic backtracking patterns:
- Nested quantifiers: `(a+)+` `(a*)*` `(a|a)*`
- Overlapping alternations: `(a|ab)+`
- User-supplied regex compiled without timeout/length limit

## 6.4 Unsafe Deserialization

| Language | Dangerous |
|---|---|
| Java | `ObjectInputStream.readObject()` on untrusted data · `XMLDecoder` · RMI |
| Python | `pickle.loads()` · `yaml.load()` (without SafeLoader) · `marshal.loads()` |
| PHP | `unserialize()` on user input · `__wakeup()`/`__destruct()` gadget chains |
| Ruby | `Marshal.load()` · `YAML.load()` (use `YAML.safe_load`) |
| JS/TS | `node-serialize` · `funcster` · `js-yaml` with unsafe schema |
| C# | `BinaryFormatter.Deserialize()` · `Json.Net` with `TypeNameHandling` |

---

# Category 7 — Configuration

## 7.1 CORS Misconfiguration

- `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`
- Origin reflected from request header without validation
- `cors({ origin: true })` (Express) — reflects any origin
- Regex bypass: `/target\.com$/` matches `attackertarget.com`

## 7.2 CSP Missing or Weak

- No CSP header set
- `unsafe-inline` or `unsafe-eval` in `script-src`
- Wildcard sources (`*.googleapis.com` may host JSONP)
- `data:` or `blob:` in `script-src`

## 7.3 Insecure Cookie Flags

- Session cookies without `HttpOnly`
- Session cookies without `Secure`
- Missing `SameSite` (or `SameSite=None` without justification)
- `Domain` set too broadly (`.example.com` when only `app.example.com` needed)

## 7.4 Debug/Dev Endpoints in Production

- `/debug/` `/actuator/` `/elmah.axd` `/_profiler/`
- `DEBUG=True` or `NODE_ENV=development` in production config
- GraphQL introspection enabled in production
- Swagger UI exposed without auth in production

## 7.5 Open Redirect

- `res.redirect(req.query.url)` without allowlist validation
- `Location` header set from user input
- `window.location = params.next` client-side without origin check

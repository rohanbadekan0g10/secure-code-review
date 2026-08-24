---
name: sast-lang-specific
description: "SAST category 10: language-specific vulnerabilities — JS/TS (prototype pollution, eval, DOM clobbering, postMessage), Python (format strings, import injection), Java (JNDI/Log4Shell, EL injection, XXE), PHP (type juggling, include vulns), Go (goroutine leaks, net/http timeouts), C# (ViewState, Path.Combine), Ruby (ERB injection, send, regex anchors)."
---

# Category 10 — Language-Specific Vulnerabilities

Load this module for any Tier 1 language. Read only the section(s) matching detected language(s).

## 10.1 JavaScript / TypeScript

**Prototype Pollution**
- `merge()` `extend()` `defaultsDeep()` with user input: `obj.__proto__.admin = true`
- `__proto__` property access on user-supplied objects
- Check: does any deep-merge/clone function receive untrusted input?

**Eval Family**
- `eval()` `Function(str)()` `setTimeout(string)` `setInterval(string)` with user data

**DOM Clobbering**
- `document.getElementById()` result used without null check in security logic
- Named form elements overriding global variables

**postMessage**
- `addEventListener('message', handler)` without origin validation in handler

**vm Module (not a sandbox)**
- `vm.runInContext()` / `vm.Script` — breakable; do not use for untrusted code isolation

**Client-Side Storage (auth tokens)**
- `localStorage.setItem('token', ...)` / `sessionStorage.setItem('token', ...)` — readable by any XSS on the same origin; flag when key name matches: `token` `jwt` `session` `auth` `access_token` `refresh_token` `apiKey`
- `window.authToken = ...` / globals storing credentials — accessible to any injected script
- Safe alternative: `HttpOnly` cookies — invisible to JavaScript even if XSS lands

## 10.2 Python

**Format String**
- `str.format()` / f-strings with user-controlled format: `"{0.__class__.__mro__}"` enables attribute traversal
- `%` formatting with user-controlled format string

**Import Injection**
- `__import__(user_input)` / `importlib.import_module(user_input)` — arbitrary code execution

**Temp File Races**
- `tempfile.mktemp()` (predictable name) — use `tempfile.mkstemp()` instead

**XML Bombs / XXE**
- `xml.etree.ElementTree.parse()` without defusing — use `defusedxml`
- `lxml` with `resolve_entities=True` (default)

## 10.3 Java

**JNDI Injection (Log4Shell pattern)**
- `InitialContext.lookup(userInput)` — CRITICAL if userInput contains `${jndi:ldap://...}`
- Any logging of user input with Log4j 2.x versions < 2.17.0

**Expression Language Injection**
- `${user_input}` evaluated in EL contexts (JSP, Thymeleaf, Spring SpEL)
- `SpelExpressionParser().parseExpression(userInput).getValue()`

**XXE**
- `DocumentBuilderFactory` without `setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`
- `SAXParserFactory` / `XMLReader` without disabling external entities

**Deserialization Gadgets**
- `ObjectInputStream.readObject()` on untrusted data
- Jackson polymorphic typing with `@JsonTypeInfo` + `enableDefaultTyping()`
- Known gadget libraries on classpath: commons-collections, Spring, Groovy

## 10.4 PHP

**Type Juggling**
- `==` in password/hash comparison: `"0e123" == "0e456"` is `true` — use `===` or `hash_equals()`
- `if ($token == $expected)` where token starts with `0e`

**Object Injection**
- `unserialize()` on user input — PHP magic methods (`__wakeup`, `__destruct`) form gadget chains

**Include Vulnerabilities**
- `include($user_input)` / `require($user_input)` with user-controlled path or URL
- `$_GET['page']` used directly in `include()`

**Variable Extraction**
- `extract($_POST)` / `extract($_GET)` — overwrites arbitrary variables including security flags

## 10.5 Go

**Goroutine Leaks**
- Goroutines spawned without context cancellation (`go func() { for { ... } }` with no `ctx.Done()`)
- HTTP handlers that never return — goroutine accumulation under load

**net/http Default Client Timeout**
- `http.DefaultClient` used without timeout: `http.Get(url)` — hangs indefinitely on slow servers
- Use `client := &http.Client{Timeout: 10 * time.Second}`

**Integer Conversion**
- `int(uint_value)` where uint could exceed `math.MaxInt` — sign flip

## 10.6 C# / .NET

**ViewState Deserialization**
- Unencrypted/unsigned ViewState — set `machineKey` with `validationAlg` and `decryptionAlg`

**LINQ Injection**
- `FromSqlRaw()` / `ExecuteSqlRaw()` with string interpolation — use `FromSqlInterpolated()`

**Regex Without Timeout**
- `new Regex(pattern)` without `RegexOptions.None, TimeSpan.FromSeconds(1)` — ReDoS risk

**Path.Combine Pitfall**
- `Path.Combine(basePath, userInput)` — if `userInput` starts with `/` or `C:\`, it IGNORES `basePath`

## 10.7 Ruby

**ERB Injection**
- `ERB.new(user_input).result` — arbitrary Ruby execution

**Dynamic Dispatch**
- `object.send(params[:method])` — invokes arbitrary methods including private ones
- `object.public_send(params[:method])` — safer but still requires allowlist

**Constant Lookup**
- `Module.const_get(params[:class])` — arbitrary class instantiation

**Regex Anchor Confusion**
- `^` and `$` in Ruby match LINE boundaries (not string boundaries)
- `\A` and `\z` match string start/end — use these for security-sensitive validation
- `format =~ /^https?:\/\//` can be bypassed with `"javascript:alert(1)\nhttps://x.com"`

## 10.8 Angular

**bypassSecurityTrust* family** — disables Angular's built-in sanitizer:
- `bypassSecurityTrustHtml(userInput)` → XSS
- `bypassSecurityTrustScript(userInput)` → script injection
- `bypassSecurityTrustUrl(userInput)` → `javascript:` protocol
- `bypassSecurityTrustResourceUrl(userInput)` → iframe/src injection
- Flag any of these receiving non-static input

**[innerHTML] binding**
- `[innerHTML]="userControlledValue"` — bypasses Angular template encoding
- Safe: `{{ userControlledValue }}` (interpolation — auto-escaped)

**Template injection**
- Dynamic component loading with user-controlled selector: `ViewContainerRef.createComponent(userInput)`

## 10.9 Vue

**v-html directive**
- `<div v-html="userContent"></div>` — equivalent to `innerHTML`, bypasses Vue's auto-escaping
- Safe: `{{ userContent }}` — auto-escaped in Vue templates
- Flag all `v-html` receiving data that originates from user input or API responses containing user data

**Dynamic component**
- `<component :is="userControlledString">` — can render arbitrary components including malicious ones

## 10.10 Next.js

**NEXT_PUBLIC_ secrets**
- Environment variables prefixed `NEXT_PUBLIC_` are bundled into the client-side JS — visible to anyone who downloads the page
- Flag: `NEXT_PUBLIC_API_SECRET`, `NEXT_PUBLIC_STRIPE_SECRET_KEY`, `NEXT_PUBLIC_DB_PASSWORD`
- Server-only secrets must use plain env vars without the `NEXT_PUBLIC_` prefix

**API routes missing auth**
- Files in `pages/api/` or `app/api/` without session/token validation at the top of the handler
- Pattern: `export default function handler(req, res) { const data = db.query(...) }` — no auth check before DB access

**getServerSideProps data leak**
- `return { props: { user: fullUserObject } }` — serializes entire server-side object to client HTML
- Flag when props include fields that shouldn't be client-visible: `password`, `hash`, `secret`, `internalId`

**Server Actions without auth**
- `'use server'` functions called from client components without verifying session at the start of the action

## 10.11 React Native

**AsyncStorage for auth tokens**
- `AsyncStorage.setItem('token', jwt)` — unencrypted, readable on rooted/jailbroken devices
- Safe alternative: `react-native-keychain` or `expo-secure-store` (uses OS keychain)
- Flag key names: `token` `jwt` `session` `auth` `apiKey` `password`

**Deep link handling without validation**
- `Linking.getInitialURL()` result used directly for navigation without scheme/host validation → open redirect on native
- `Linking.addEventListener('url', handler)` without validating the URL before routing

**Insecure fetch config**
- `fetch(url, { ... })` — React Native bundles do not enforce certificate pinning by default
- Flag for financial/health apps: no `react-native-ssl-pinning` or equivalent

## 10.12 Flutter / Dart

**SharedPreferences for sensitive data**
- `prefs.setString('token', jwt)` — unencrypted on Android (XML file in app data)
- Safe: `flutter_secure_storage` (uses Android Keystore / iOS Keychain)
- Flag key names: `token` `jwt` `session` `password` `apiKey`

**HTTP without certificate pinning**
- `http.get(Uri.parse(url))` — no certificate pinning in financial/health apps
- Flag absence of `SecurityContext` with pinned certificates for sensitive endpoints

**Deep link intent validation**
- `onGenerateRoute` receiving external URIs without host/scheme validation
- `AndroidManifest.xml`: `<intent-filter>` with `android:autoVerify` not set to `true` for App Links

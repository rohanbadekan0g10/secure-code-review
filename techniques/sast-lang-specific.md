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

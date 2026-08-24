---
name: sast-injection
description: "SAST category 1: injection sink analysis — SQL injection, command injection, XSS, SSTI, LDAP, XPath, NoSQL injection, and CRLF/header injection. Trace user input to dangerous sinks without sanitization."
---

# Category 1 — Injection Sinks

Trace: does user input (`req.body`, `req.query`, `req.params`, form data, URL) reach any sink below without parameterization/encoding?

## 1.1 SQL Injection

| Language | Dangerous sinks | Safe alternative |
|---|---|---|
| JS/TS | String concat/template in `db.query()` `knex.raw()` `sequelize.query()` `pool.query()` | Parameterized `$1`/`?` placeholders |
| Python | f-string in `cursor.execute()` `engine.execute()` Django `extra()`/`raw()` SQLAlchemy `text()` | `%s` with params tuple |
| Java | String concat in `Statement.execute()` `createQuery()` `JdbcTemplate.query()` | `PreparedStatement` named params |
| PHP | Interpolation in `mysqli_query()` `PDO::query()` `DB::select()` `whereRaw()` | `PDO::prepare()` Eloquent builder |
| C# | String concat in `SqlCommand` `FromSqlRaw()` `ExecuteSqlRaw()` | `SqlParameter` `FromSqlInterpolated()` LINQ |
| Go | `fmt.Sprintf` in `db.Query()` `db.Exec()` GORM `Raw()` | `db.Query(sql, args...)` |
| Ruby | Interpolation in `connection.execute()` `find_by_sql` `where("name = '#{}'")` | `where(name: value)` `where("name = ?", val)` |

## 1.2 Command Injection

| Language | Dangerous |
|---|---|
| JS/TS | `exec()` `execSync()` `spawn(shell:true)` backtick in exec |
| Python | `os.system()` `os.popen()` `subprocess.call(shell=True)` `eval()` `exec()` |
| Java | `Runtime.exec()` `ProcessBuilder` with user input `ScriptEngine.eval()` |
| PHP | `exec()` `system()` `passthru()` `shell_exec()` `popen()` backtick |
| C# | `Process.Start()` with user input in arguments |
| Go | `exec.Command()` with user input |
| Ruby | `system()` backtick `%x{}` `Kernel.exec()` `IO.popen()` |

## 1.3 XSS (Output Sinks)

| Context | Dangerous |
|---|---|
| React | `dangerouslySetInnerHTML` `innerHTML` `document.write()` |
| Express/Node | `res.send(userInput)` unescaped template output (`<%- %>` `{{{ }}}` `!{var}`) |
| Django | `mark_safe()` `|safe` `{% autoescape off %}` on user data |
| Flask/Jinja2 | `Markup()` `|safe` `{% autoescape false %}` on user data |
| Java JSP | `<%= var %>` `th:utext` `out.println()` |
| PHP | `echo $var` without `htmlspecialchars()` `{!! $var !!}` `|raw` |
| C# Razor | `@Html.Raw()` |
| Go | `template.HTML()` cast or `text/template` instead of `html/template` |
| Ruby ERB | `raw()` `html_safe` on user input `<%== %>` |

## 1.4 SSTI (Template Injection)

User input passed directly to template rendering:
- Flask: `render_template_string(user_input)`
- Jinja2: `Template(user_input).render()`
- Ruby: `ERB.new(user_input).result`
- JS: `new Function(user_input)`
- Java: Freemarker/Velocity/Thymeleaf with user-controlled template string

## 1.5 LDAP Injection
User input in LDAP filter strings: `(&(uid=USER_INPUT)...)` without `ldap_escape_filter()`.

## 1.6 XPath Injection
User input in XPath: `//user[name='USER_INPUT']` string construction, `XPathExpression.evaluate()` with concat.

## 1.7 NoSQL Injection
- JS/TS MongoDB: `collection.find({field: req.body.value})` where value could be `{$gt: ""}` or `$where` with string
- Python PyMongo: `collection.find({"field": request.json["value"]})` without type enforcement

## 1.8 CRLF / Header Injection
User input in HTTP response headers without CRLF stripping:
- `res.setHeader('Location', userInput)` → `\r\n` splits into new headers
- `response.addHeader("X-Custom", userInput)`

## Finding Format

```json
{
  "id": "SF01", "category": "injection", "class": "sql_injection",
  "severity": "CRITICAL", "confidence": "HIGH",
  "file": "src/services/userService.js", "line": 47,
  "sink": "db.query(sql)", "source": "req.body.username",
  "data_flow": ["req.body.username → controller:14", "→ service:28", "→ db.query():47 (SINK)"],
  "route": "POST /api/users", "remediation": "Use parameterized query"
}
```

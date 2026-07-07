# web security notes

## XSS

basic reflected - put payload in param, see if it echoes back unescaped
`<script>alert(1)</script>` rarely works now but worth trying
better: `"><img src=x onerror=alert(1)>`

stored XSS is way more impactful, look for comment sections, profile fields, anything that gets saved and displayed to other users

DOM XSS - look at JS source, find sinks like `innerHTML`, `document.write`, `eval`
sources: `location.hash`, `location.search`, `document.referrer`

bypass filters:
- `<ScRiPt>` case mixing
- `<svg/onload=alert(1)>`
- `javascript:alert(1)` in href sometimes still works on older apps
- `<details open ontoggle=alert(1)>`

CSP bypass - if they allow `unsafe-inline` its useless, check for JSONP endpoints that can be abused

---

## SQLi

manual test: add `'` see if error, then `' OR '1'='1`
error based vs blind vs time based

time based example (mysql):
`' AND SLEEP(5)--`

sqlmap is fine for confirming but learn to do it manually first or you won't understand what's happening

union based:
1. find number of columns: `ORDER BY 1--`, `ORDER BY 2--` etc until error
2. `UNION SELECT NULL,NULL,NULL--` match the columns
3. find which column is reflected: `UNION SELECT 'a',NULL,NULL--`

nosql injection - different syntax, mongodb uses `{"$gt":""}` type stuff

---

## IDOR

change IDs in requests - user_id=123 -> try 122, 124
not always sequential, sometimes UUIDs -> look for other places where IDs leak (logs, referrer headers, other API responses)

horizontal vs vertical:
- horizontal = same privilege, different user
- vertical = escalate to higher privilege role

look in: GET params, POST body, cookies, JWT payload, path params `/api/users/123/profile`

---

## SSRF

classic: `url=http://169.254.169.254/latest/meta-data/` (AWS metadata)
GCP: `http://metadata.google.internal/computeMetadata/v1/`

bypasses when they block IPs:
- use DNS rebinding
- `http://[::ffff:169.254.169.254]/`
- decimal encoding: 169.254.169.254 -> 2852039166
- `http://0x7f000001/` hex

blind SSRF - use burp collaborator or interactsh, look for OOB interactions even if no response

also check: webhook URLs, PDF generators, image fetchers, anything that makes outbound requests

---

## file upload

check: can you upload .php, .phtml, .phar?
if extension blocked try double extension: `shell.php.jpg`
check MIME type only? change Content-Type to image/jpeg with php content
null byte: `shell.php%00.jpg` (old PHP)

where does it get stored? can you access it directly?

---

## headers to always check

- `X-Forwarded-For` - can sometimes bypass IP restrictions
- `Referer` - some apps trust this for CSRF or access control
- `Host` - host header injection -> password reset poisoning
- `X-Original-URL` / `X-Rewrite-URL` - path bypass

---

## misc

CORS misconfiguration: `Origin: https://evil.com` in request, if response has `Access-Control-Allow-Origin: https://evil.com` with credentials -> big deal

clickjacking - if no X-Frame-Options or CSP frame-ancestors, site can be iframed

HTTP request smuggling - CL.TE or TE.CL, more complex, look into it more later

open redirect: `?redirect=https://evil.com` - useful for phishing or sometimes OAuth bypasses

---

## stuff i keep forgetting

- always check source code, comments have juicy stuff sometimes
- robots.txt, sitemap.xml for hidden paths
- check old JS files in Wayback Machine
- `.git` folder exposed = game over, use git-dumper
- error messages often reveal framework, version, server type

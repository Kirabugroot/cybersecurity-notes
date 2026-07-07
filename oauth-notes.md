# oauth / auth notes

## oauth 2.0 basics

flows:
- authorization code - most common, server side apps, has PKCE now
- implicit - deprecated but still in the wild, token in URL fragment (bad)
- client credentials - machine to machine, no user
- resource owner password - legacy, avoid

the flow:
1. app redirects user to auth server with `client_id`, `redirect_uri`, `scope`, `state`
2. user logs in and approves
3. auth server redirects back with `code`
4. app exchanges `code` for `access_token` (server to server, includes `client_secret`)
5. app uses token to hit API

---

## attack vectors

### redirect_uri manipulation
if not validated strictly:
- `redirect_uri=https://attacker.com`
- path traversal: if `https://app.com/callback` is whitelisted, try `https://app.com/callback/../attacker`
- subdomain: `https://evil.app.com` if they whitelist `*.app.com`

if you steal the code, you can exchange it for a token (need client_secret usually, but some flows don't)

### state param
`state` should be random and tied to session - CSRF protection
if missing or predictable -> CSRF on oauth flow -> attacker can link their account to victim's

### token leakage
- referrer header leaks token if it's in URL (implicit flow)
- browser history
- logs

### scope manipulation
change scope param to request more permissions than intended
`scope=email` -> try `scope=email profile admin`
app might not validate what it asked for vs what it gets back

### open redirect + oauth
if app has open redirect, can be used to leak auth code via referrer

### PKCE
meant to secure auth code flow for public clients (SPAs, mobile apps)
if PKCE not enforced server side, it's just cosmetic

---

## JWT stuff

header.payload.signature - base64 encoded, dot separated

common vulns:
- `alg: none` - remove signature, set alg to none, server accepts it (old/broken libs)
- RS256 -> HS256 confusion: if server uses public key as HMAC secret when you switch alg
- weak secret -> crack with hashcat: `hashcat -a 0 -m 16500 token.txt wordlist.txt`
- kid injection: if kid param used in SQL query or path
- `jku`/`x5u` header - server fetches key from URL, point to your own key

always check payload for sensitive info, sometimes has roles, user id, internal paths

decode without verifying: `echo "payload" | base64 -d`

---

## session management

- session fixation: if session ID doesn't change after login
- check if session ID is in URL (bad, logs everything)
- httponly? secure flag? samesite?
- logout: does it actually invalidate server side or just delete cookie client side?

---

## password reset

- token in URL? check referrer leakage to third party scripts
- token expiry? some never expire
- host header injection: if app uses Host header to build reset URL -> `Host: attacker.com`
- token reuse? same token for multiple requests?
- brute force? rate limiting on the endpoint?

---

## notes to self

- always test both the "happy path" and weird edge cases in oauth
- when testing, use two browsers - one as attacker, one as victim
- intercept ALL oauth requests, the devil is in the details
- check `.well-known/openid-configuration` for discovery doc, reveals endpoints

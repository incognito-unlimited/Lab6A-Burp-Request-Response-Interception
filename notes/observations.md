# Lab 6A — Sanitized Observations

Target: `http://127.0.0.1:8081` (Docker container lab6a-dvwa, image sagikazarmark/dvwa:latest, port 127.0.0.1:8081 -> 80)
Burp proxy: `http://127.0.0.1:8080` · Security level: `impossible`

## Scenario 1 — Hidden form field

| Item | Observation |
|---|---|
| Route | POST /login.php (DVWA login form) |
| Hidden field | `user_token` (anti-CSRF token; live value redacted) |
| Original | HTTP 302 -> `index.php` |
| Modified | `user_token=INVALID_LAB_TEST_TOKEN` -> HTTP 302 -> `login.php` |
| Result | Rejected with `CSRF token is incorrect`. |
| Conclusion | Server-side validation enforced. Hidden field is not a security boundary. |

## Scenario 2 — Session cookie

| Item | Observation |
|---|---|
| Route | GET /vulnerabilities/fi/?page=file3.php |
| Cookie | `PHPSESSID` (value never published) |
| Baseline | HTTP 200, authenticated: true |
| Modified | `PHPSESSID=INVALID_LAB_TEST_SESSION` |
| Response | HTTP 302 -> `../../login.php` |
| Conclusion | Invalid identifier rejected; server failed closed to login. |

## Scenario 3 — Request header

| Item | Observation |
|---|---|
| Route | GET /vulnerabilities/fi/?page=file3.php |
| Header | `X-Forwarded-For` |
| Baseline | HTTP 200; XFF reflected: false |
| Modified | `X-Forwarded-For: 192.0.2.10` |
| Response | HTTP 200; XFF reflected: true |
| Conclusion | Informational reflection only; no access-control decision observed. |

## Limitations

- HTTPS interception was configured, but the target is HTTP-only; no TLS exchange was available.
- Header test covers one route and does not prove a trust-boundary vulnerability.
- No credentials, live cookies, or live tokens are stored in this repository.

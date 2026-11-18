# Lab 6A — Burp Suite Request/Response Interception and Manipulation

This repository contains sanitized evidence for Lab 6A (Experiment 7): intercepting
and modifying HTTP requests with Burp Suite Community Edition against a local,
intentionally vulnerable DVWA instance.

## Scope and ethical boundary

- **Target:** `http://127.0.0.1:8081` only.
- **Deployment:** Docker container lab6a-dvwa, image sagikazarmark/dvwa:latest, port 127.0.0.1:8081 -> 80.
- **Network:** loopback-only Docker port publish; no public target.
- **Scope in Burp:** exact origin `http://127.0.0.1:8081` only.
- **No public websites, university systems, third-party accounts, or external targets** were used.
- A disposable DVWA test account was used. Credentials, live cookies, and live tokens are redacted everywhere.
- No session was stolen, replayed, or forged; only an explicitly invalid test cookie was submitted.

## Environment

- Burp Suite Community Edition 2026.11.1, Proxy listener `127.0.0.1:8080`.
- Firefox 160.0 dedicated lab profile with manual HTTP/HTTPS proxy `127.0.0.1:8080`
  and loopback hijacking enabled. No normal browser profile was changed.
- Captured traffic was produced by a local test client routed through the same
  loopback Burp listener.
- Burp CA certificate downloaded from `http://burp/cert` for optional HTTPS
  interception. It was not installed in any browser store because the target is
  HTTP-only and no TLS exchange was available.
- DVWA security level: `impossible`.
- Target is HTTP-only; HTTPS interception was configured but not exercised against live TLS traffic.

## Tests performed

1. **Hidden form field** — `POST /login.php`, hidden `user_token` anti-CSRF field.
   - Original: HTTP 302 -> `index.php`.
   - Modified: `user_token=INVALID_LAB_TEST_TOKEN` -> HTTP 302 -> `login.php`.
   - The redirected page displayed `CSRF token is incorrect`. Server-side validation held.

2. **Session cookie** — `GET /vulnerabilities/fi/?page=file3.php`, `PHPSESSID`.
   - Baseline: HTTP 200, authenticated page rendered.
   - Modified: `PHPSESSID=INVALID_LAB_TEST_SESSION` -> HTTP 302 -> `../../login.php`.
   - Server failed closed to login. No valid session value is stored in this repository.

3. **Request header** — `GET /vulnerabilities/fi/?page=file3.php`, `X-Forwarded-For`.
   - Baseline: no XFF header; no `Forwarded for` line in the response.
   - Modified: `X-Forwarded-For: 192.0.2.10` -> HTTP 200;
     value reflected in the response body.
   - Reflection only; no access-control decision or authentication state changed.

## Repository contents

- `README.md` — this file.
- `report/` — populated DOCX and exported PDF.
- `evidence/` — genuine, redacted screenshots from the lab run.
- `requests/` — sanitized request/response text captured through Burp.
- `notes/observations.json` and `notes/observations.md` — structured observations.
- `.gitignore` — excludes browser profiles, Burp project data, certificates, keys, and office lock files.

## Limitations

- The isolated DVWA target is HTTP-only, so TLS interception was configured but could not be
  verified against a live HTTPS exchange. Consequently the downloaded Burp CA was not
  installed in any browser certificate store.
- Header testing covered one reflective route; the result does not establish a trust-boundary
  vulnerability or an access-control bypass.
- Findings apply only to the local lab instance.

## Reproducing

Start the DVWA container, start Burp with the local-only listener, launch the dedicated
Firefox profile, add only `http://127.0.0.1:8081` to Burp scope, then follow the numbered
steps in the report. Saved request/response files are already sanitized and can be reviewed
without handling live session material.

# API Security Risk Analysis: Findings

**Targets:** JSONPlaceholder (tested 18 Nov 2026) and Restful-Booker (tested 19 Nov 2026)
**Report date:** 21 November 2026
**Assessor:** Sukh Ram Godara
**Program:** Future Interns, Cyber Security Internship, Task 3
**Approach:** Read-only requests and documentation review. No login, no PUT/PATCH/DELETE, no flooding.

**Tools:** Postman, curl, Restful-Booker documentation

Both APIs are demos with fake data. Risks are rated as if they held real customer data. Names such as API2 refer to the OWASP API Security Top 10 (2023).

## Summary

| Risk | JSONPlaceholder | Restful-Booker | Total |
|---|---|---|---|
| High | 2 | 2 | 4 |
| Medium | 3 | 1 | 4 |
| Low | 3 | 4 | 7 |
| Informational | 1 | 1 | 2 |
| **Total** | **9** | **8** | **17** |

## JSONPlaceholder

| ID | Risk | Finding | What it means (plain language) | Recommended fix | Evidence |
|---|---|---|---|---|---|
| JP-01 | High | No authentication required (API2) | Anyone on the internet can read user and post data without logging in | Require a valid token or key on every endpoint that returns personal data | `01-jp-users1-full-record.png` |
| JP-02 | High | Excessive data exposure (API3) | One response returns name, email, full address, GPS location, phone and company | Return only the fields each client needs | `01-jp-users1-full-record.png` |
| JP-03 | Medium | Sequential IDs, no per-record access check (API1) | Changing the ID (`users/1`, `users/2` ...) shows other records, so all records can be collected one by one | Check access per record, use non-guessable IDs, add pagination | `02-jp-users2-other-record.png` |
| JP-04 | Medium | CORS allows any origin with credentials (API8) | A foreign origin was echoed back with credentials allowed, so any website could read responses together with login cookies | Allow only a fixed list of trusted origins | `03-jp-cors-origin-echoed.png` |
| JP-05 | Medium | No input validation on create (API8) | An empty request was accepted with 201 Created | Validate required fields, types and length; return 400 for invalid input | `04-jp-post-empty-201.png` |
| JP-06 | Low | Technology disclosed (`x-powered-by: Express`) | Tells attackers which framework runs the API | Remove the header | `05-jp-response-headers.png` |
| JP-07 | Low | Very high rate limit (3,600,000) (API4) | A limit this high does not stop automated data collection | Set realistic limits per user and IP | `05-jp-response-headers.png` |
| JP-08 | Low | `Strict-Transport-Security` header missing (API8) | Browsers are not told to always use HTTPS | Add HSTS | `05-jp-response-headers.png` |
| JP-09 | Info | Responses cacheable for 12 hours | Fine for public data, risky for private data | Use `no-store` for private responses | `05-jp-response-headers.png` |

## Restful-Booker

| ID | Risk | Finding | What it means (plain language) | Recommended fix | Evidence |
|---|---|---|---|---|---|
| RB-01 | High | Booking records readable without authentication (API2, API3) | Guest name, price, deposit status, dates and special needs can be read by anyone | Require authentication; return only needed fields | `07-rb-booking1-no-auth.png`, `14-rb-docs-get.png` |
| RB-02 | Medium | Full booking ID list in one response (API1, API4) | One request returns every booking ID (10,568 bytes at test time, larger in an earlier run), which makes collecting records easy | Paginate and limit list endpoints, use non-guessable IDs | `08-rb-booking-list-size.png` |
| RB-03 | High | Default credentials published in documentation (API2) | The documentation shows a ready-to-use username and password, which could give a token for changing and deleting bookings | Remove default credentials, use unique strong secrets, rotate them | `11-rb-docs-auth.png` |
| RB-04 | Low | Basic authentication and cookie token supported (API2) | Basic auth sends the username and password (base64 is not encryption) with every request; the token travels in a Cookie header | Use short-lived tokens in the Authorization header over HTTPS | `12-rb-docs-update.png`, `13-rb-docs-delete.png` |
| RB-05 | Low | Security headers missing (API8) | No Strict-Transport-Security, X-Content-Type-Options or Cache-Control header | Add the standard security headers | `09-rb-ping-headers.png` |
| RB-06 | Low | Technology disclosed (API8) | Headers show Express, Heroku and heroku-router | Remove or generalise these headers | `09-rb-ping-headers.png` |
| RB-07 | Low | No rate-limit information observed (API4) | No rate-limit headers appeared; flooding was not tested, so limits may exist but are not visible | Apply and document rate limits | `09-rb-ping-headers.png` |
| RB-08 | Info | Unusual status codes | `GET /ping` returned 201 Created, and the documentation says DELETE returns 201 Created | Use 200 for reads and a delete-appropriate code for deletes | `09-rb-ping-headers.png`, `13-rb-docs-delete.png` |

## Authentication Analysis (Restful-Booker, from documentation)

| Item | What the documentation shows |
|---|---|
| Token endpoint | `POST /auth` with a JSON body containing username and password |
| Default credentials | Documentation lists default values for username and password |
| How the token is sent | In a Cookie header, or as an Authorization header using Basic authentication |
| Needs a token | PUT and DELETE on `/booking/:id` |
| No token needed | `GET /ping`, `GET /booking`, `GET /booking/:id` (confirmed by tests) |
| Not tested | Real login, token lifetime, cookie flags, and PUT or DELETE without a token (these would change data) |

## Positive Observations

- Both APIs use HTTPS.
- JSONPlaceholder returns a short 404 for invalid IDs (`abc`, `-1`, `99999`) with no stack trace (`06-jp-invalid-ids-404.png`).
- Restful-Booker did not return a CORS allow header for a foreign origin (`10-rb-cors-not-echoed.png`).
- Per its documentation, Restful-Booker requires a token for PUT and DELETE.

## Fix Priority

1. Protect data behind authentication and remove published default credentials (JP-01, RB-01, RB-03)
2. Limit what each caller can see: fewer fields, per-record access checks, non-guessable IDs, pagination (JP-02, JP-03, RB-02)
3. Fix CORS: allow only trusted origins (JP-04)
4. Validate input on create endpoints (JP-05)
5. Harden headers and limits: security headers, remove technology banners, realistic rate limits, short-lived tokens (JP-06 to JP-08, RB-04 to RB-07)

## Notes

- Risk ratings are the assessor's classification.
- Guest names in screenshots were hidden.
- Restful-Booker is a shared playground, so list sizes change over time.

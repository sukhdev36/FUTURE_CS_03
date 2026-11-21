# FUTURE_CS_03: API Security Risk Analysis

**Future Interns | Cyber Security Internship | Task 3**

**Author:** Sukh Ram Godara

**Report date:** 21 November 2026

## Overview

This repository contains a read-only API security risk analysis of two public demo APIs. The goal was to identify common API risks (open endpoints, excessive data exposure, weak authentication, missing access control, rate limiting and input validation), rate each risk as Low, Medium or High, explain the business impact in simple language, and suggest fixes.

**Final report:** [`report/API_Security_Risk_Analysis_Report.pdf`](report/API_Security_Risk_Analysis_Report.pdf)

## APIs Tested

| API | Tested on | Why this target |
|---|---|---|
| JSONPlaceholder (`jsonplaceholder.typicode.com`) | 18 Nov 2026 | Public fake API built for testing and learning |
| Restful-Booker (`restful-booker.herokuapp.com`) | 19 Nov 2026 | Public practice API for learning API testing |

Both APIs hold fake data, so risks are rated **as if they held real customer data**.

## Scope

**In scope**
- Read-only GET requests
- One safe POST with an empty body on JSONPlaceholder (it only simulates creating data)
- Header and response inspection, and one foreign-Origin request to check CORS
- Documentation-based review of Restful-Booker authentication

**Not performed**
- Exploitation or bypass attempts, brute force, flooding or denial of service
- Login or token requests, and PUT, PATCH or DELETE requests
- Injection or attack payloads
- Testing any private or production API

See [`scope.txt`](scope.txt).

## Methodology

| Risk area | How it was checked (read-only) |
|---|---|
| Open or unauthenticated endpoints | GET requests sent without any token or key |
| Excessive data exposure | Compared fields returned with what a client would need |
| Weak or missing tokens | Reviewed how tokens are issued and sent, using the documentation (no login was made) |
| Authorization (other users' data) | Changed the ID in the address (for example `users/1` and `users/2`) |
| Missing rate limiting | Looked at rate-limit headers only, no flooding |
| Input validation | Sent invalid IDs and one empty POST body |
| Browser access rules (CORS) | One request with a foreign `Origin` header |

Names such as API1 or API2 in the findings refer to the OWASP API Security Top 10 (2023).

## Tools Used

- Postman
- curl (Kali Linux)
- Restful-Booker API documentation
- Canva for report design

## Summary of Results

| Risk | Count |
|---|---|
| High | 4 |
| Medium | 4 |
| Low | 7 |
| Informational | 2 |
| **Total** | **17** |

**Key findings**
- Data can be read without logging in on both APIs.
- Records can be collected one by one because IDs are sequential, and Restful-Booker lists every booking ID in one response.
- One JSONPlaceholder record returns name, email, full address, GPS location and phone.
- The Restful-Booker documentation publishes default credentials.
- JSONPlaceholder echoes any website origin with credentials allowed, and accepts an empty create request.

Full details, business impact and fixes are in [`findings.md`](findings.md) and the PDF report.

## Repository Structure

```
FUTURE_CS_03/
├── README.md
├── scope.txt
├── findings.md
├── report/
│   └── API_Security_Risk_Analysis_Report.pdf
└── evidence/
    └── 01-... to 14-... (Postman, curl and documentation screenshots)
```

## Limitations

- Both APIs are demos with fake data, so ratings assume real data.
- Authentication was reviewed from documentation only. No login, token request, PUT or DELETE was made.
- Rate limiting was judged from headers only.
- Restful-Booker is a shared playground, so list sizes change over time.
- Classifications are the author's judgement based on the available evidence.

## Ethics Statement

All testing was read-only, on public demo APIs built for testing. No credentials were used, no data was modified, and no exploitation was attempted. Guest names in screenshots were hidden.

## Disclaimer

This work was done for learning purposes as part of the Future Interns Cyber Security internship. The findings describe demo APIs and do not represent any real organisation.

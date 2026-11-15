# Findings — SecDojo Comepete

This document records the security weaknesses confirmed during the authorized challenge.

| ID | Finding | Evidence from the lab report | Security consequence |
|---|---|---|---|
| F-01 | Directory listing | `/data/` exposed `auth.db` and JSON application stores | Unauthenticated information disclosure |
| F-02 | Public database access | `auth.db` was downloadable over HTTP | Disclosure of authentication records |
| F-03 | Plaintext credentials | The `admins` table contained a plaintext password field and value | Administrator account compromise |
| F-04 | Local File Inclusion | The `file` parameter accepted a PHP stream-wrapper path | PHP source disclosure |
| F-05 | Unsafe template evaluation | `SimpleSmarty::fetch()` used a regex callback and `eval()` on template content | PHP execution in the web process |
| F-06 | Web-served writable data | Quiz descriptions were persisted in `data/quizzes.json`, which was also publicly retrievable | Exposed application data and observable output |

## Confirmed outcome

- Execution identity: `www-data` (UID 33).
- Working directory: `/var/www/html`.
- Flag file: `/home/local.txt`.
- Result: challenge flag captured.

Passwords, session cookies, operator IP details and reusable execution payloads are excluded from this public repository.

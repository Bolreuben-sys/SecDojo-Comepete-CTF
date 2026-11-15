# SecDojo — Comepete

**PHP Web Exploitation | Information Disclosure, Local File Inclusion (LFI) and Remote Code Execution (RCE)**

> Authorized CTF/lab exercise. This repository documents the vulnerability chain and the lessons learned. Target addresses, recovered credentials, session identifiers, and reusable command-execution payloads have intentionally been omitted from this public write-up.

## Overview

Comepete is a PHP-based certification quiz application hosted on Apache. During the authorized SecDojo challenge, several weaknesses could be chained to obtain administrative access and execute PHP code in the web application's context.

The successful path was:

1. HTTP service discovery identified the web application and an exposed `/data/` directory.
2. Directory listing and direct access exposed application data, including the SQLite authentication database.
3. Inspection of the database showed that administrator credentials were stored in plaintext.
4. The application's `file` parameter was vulnerable to Local File Inclusion (LFI). PHP's `php://filter` wrapper could be used to disclose application source code.
5. Review of the administrative source revealed a custom template processor that passed user-controlled template content to PHP `eval()`.
6. The quiz-description processing path therefore provided code execution in the web-server process context.
7. The challenge flag was retrieved from the lab's designated flag file.

**Result:** Successful completion. The confirmed execution context was `www-data` (UID 33).

## Lab details

| Item | Value |
|---|---|
| Platform | SecDojo |
| Challenge | Comepete |
| Application | Certification Quizzes (PHP) |
| Web server | Apache 2.4.52 (Ubuntu) |
| Web service | HTTP, TCP/80 |
| Operating system | Ubuntu |
| Execution context | `www-data` (UID 33) |
| Web root | `/var/www/html` |
| Flag file | `/home/local.txt` |
| Outcome | Flag captured |

Live host addresses and operator-specific network details are intentionally excluded from this public copy.

## Vulnerability chain

```text
HTTP service discovery
        |
        v
Exposed /data/ directory
        |
        v
Publicly accessible SQLite database
        |
        v
Plaintext administrator credential disclosure
        |
        v
LFI through the application's file parameter
        |
        v
PHP source-code disclosure
        |
        v
Unsafe template processing using eval()
        |
        v
PHP execution as www-data
        |
        v
CTF flag retrieval
```

## 1. Reconnaissance

The initial service discovery identified the HTTP service and useful application paths.

Example command (replace the placeholder with the address of your authorized lab target):

```bash
nmap -Pn -p22,80 --script http-title,http-headers,http-enum <LAB_TARGET>
```

The scan identified:

- SSH on TCP/22.
- HTTP on TCP/80.
- An application titled **Certification Quizzes - Secdojo**.
- A login page at `/login.php`.
- A browsable `/data/` directory.

The application redirected the root request to a PHP page selected through a `file` parameter. This parameter became an important area for further investigation.

## 2. Information disclosure — exposed application data

The `/data/` directory allowed directory listing. The report identified the following files:

- `auth.db`
- `quizzes.json`
- `results.json`
- `users.json`

The authentication database was accessible without authentication. This exposed internal application data that should not have been served directly by the web server.

**Security impact:** An unauthenticated visitor could retrieve application data, including the database used for authentication.

## 3. Plaintext credential storage

The exposed database was an SQLite database containing an `admins` table. Its schema stored a username and password in ordinary text fields.

The recovered administrator credential was sufficient to authenticate to the administrative interface. The actual password has been removed from this public write-up.

**Security impact:** Anyone able to download the database could obtain administrator credentials without password cracking.

No SQL injection was required for this part of the challenge.

## 4. Local File Inclusion — PHP source disclosure

The application used a request parameter named `file` to select PHP content for inclusion. The parameter was not adequately restricted to approved application pages.

The PHP `php://filter` stream wrapper, with Base64 encoding, made it possible to disclose PHP source rather than simply render the included page.

The source review covered the application's login, quiz and administrative logic. The most important discovery was in the administrative quiz-description processing path.

**Security impact:** Source disclosure exposed application internals, authentication logic, file paths and the unsafe template-processing implementation.

## 5. Root cause — unsafe template evaluation

The administrative source contained a custom `SimpleSmarty` implementation. It searched template content for PHP-style markers and passed the matched content to `eval()`.

The relevant behavior was:

- User-controlled quiz descriptions were passed to the template processor.
- A regular-expression callback extracted content between the template markers.
- The extracted content was evaluated as PHP.
- Output buffering captured the evaluated output.
- The resulting description was stored in `data/quizzes.json`.

This converted a quiz-description field into an unsafe PHP execution path.

**Security impact:** An authenticated administrator could cause arbitrary PHP code to execute with the privileges of the web-server process. In this challenge, execution was confirmed as `www-data` (UID 33).

The public version intentionally explains the vulnerable code path without including a reusable command-execution payload or an automated exploit script.

## 6. Flag capture

After confirming the execution context, the challenge's designated flag file was identified as:

```text
/home/local.txt
```

The flag was successfully retrieved during the authorized lab exercise.

**Challenge result:** `flag_4f84a60a_fb19_43c8_83ed_85e27b447472`

## Findings summary

| # | Finding | Affected component | Impact |
|---:|---|---|---|
| 1 | Directory listing enabled | `/data/` | Exposes application files |
| 2 | Authentication database publicly accessible | `auth.db` | Enables credential disclosure |
| 3 | Plaintext password storage | `admins` table | Administrator account compromise |
| 4 | Local File Inclusion | `file` parameter | PHP source-code disclosure |
| 5 | Unsafe use of `eval()` | Custom template processor | PHP code execution |
| 6 | Sensitive data stored in web-served directory | `data/` | Exposes application data and creates an output channel |

## Remediation

| Priority | Recommendation |
|---|---|
| Critical | Remove `eval()` from template processing. Never execute user-controlled content as PHP. |
| Critical | Move databases and application data outside the web root. |
| Critical | Disable directory listing in Apache using `Options -Indexes`. |
| High | Hash passwords with PHP `password_hash()` and verify them with `password_verify()`. |
| High | Replace user-controlled include paths with a strict allow-list of permitted page names. |
| High | Validate resolved file paths and ensure they remain within the intended application directory. |
| High | Prevent direct HTTP access to SQLite databases, JSON stores, configuration files and other sensitive files. |
| Medium | Rotate exposed credentials and invalidate existing sessions. |
| Medium | Run the web application with least privilege and restrict access to unrelated host files. |
| Medium | Monitor sensitive application data for unexpected changes and access. |

## Key lessons learned

- Directory listing can expose more than filenames; it can expose an application's most sensitive data.
- Plaintext password storage turns a file disclosure into an account-compromise incident.
- LFI can disclose source code, revealing weaknesses that are otherwise difficult to identify externally.
- Template engines must never evaluate untrusted input as executable code.
- An authenticated application feature can become a serious RCE vector when unsafe evaluation is involved.
- Defense in depth matters: fixing any one weakness in the chain reduces the likelihood of a complete compromise.

## Tools used

- Nmap
- cURL
- SQLite
- Python 3
- Base64 utilities
- Kali Linux

## Repository structure

```text
comepete-secdojo/
├── README.md
├── FINDINGS.md
├── REMEDIATION.md
├── TIMELINE.md
└── evidence/
    └── README.md
```

## Disclaimer

This material is for education, defensive security learning and documentation of an authorized CTF/lab exercise. Do not test these techniques against systems without explicit permission.

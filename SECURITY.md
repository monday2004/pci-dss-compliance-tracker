SECURITY.md — PCI-DSS Compliance Tracker (Tool-17)

Sprint: 14 April – 9 May 2026
Team: 7 Members
Security Reviewer: Rakshith
Last Updated: 5 May 2026
Status: Final — All major issues addressed

---

Executive Summary

This project deals with compliance and audit-related data, so security was treated as an important part of development from the beginning.

Instead of waiting until the end, security checks were done alongside development. As features were built, they were tested using Postman, manual attack inputs, and later validated using OWASP ZAP scans.

By the end of the review, all critical and high severity issues were resolved. A few low-level risks remain, but they are documented and acceptable for the current scope of the project.

---

1. Threat Model — Based on OWASP Top 10

1.1 Broken Access Control

Scenario tested:
A VIEWER user tries to call a DELETE API directly using Postman.

Implementation:

- Spring Security with role-based access control
- @PreAuthorize used on restricted endpoints
- Roles: ADMIN > MANAGER > VIEWER
- JWT contains role information

Result:
VIEWER attempting DELETE -> 403 Forbidden (verified manually)

---

1.2 Cryptographic Failures

Scenario tested:
Checking if passwords or secrets are exposed or stored insecurely.

Implementation:

- Passwords hashed using BCrypt
- JWT signed using secret from environment variables
- No secrets stored in code or logs

Verification:

- Database contains only hashed passwords
- No API keys or secrets found in repository history

---

1.3 Injection (SQL + Prompt Injection)

SQL Injection testing:

- Payloads like ' OR 1=1 -- tested using Postman
- Queries handled using JPA (no raw SQL)

Prompt Injection testing:

- Inputs like “Ignore previous instructions…” tested on AI endpoints

Implementation:

- Input validation in backend (DTO level)
- Sanitisation in AI service
- Suspicious patterns blocked before processing

Result:
Malicious inputs return 400 Bad Request

---

1.4 Insecure Design

Scenario tested:
Heavy API /generate-report being spammed.

Implementation:

- Rate limiting applied (10 requests/min)
- Async processing used for report generation
- Redis caching for repeated requests

Result:
Excess requests -> 429 Too Many Requests

---

1.5 Security Misconfiguration

Scenario tested:
Checking for exposed stack traces and missing headers.

Implementation:

- Disabled stack traces in responses
- Added security headers:
  - X-Content-Type-Options
  - X-Frame-Options
- Restricted Swagger in production
- .env excluded from Git

Verification:
Error responses return clean JSON, no internal details exposed

---

1.6 Vulnerable Dependencies

Check performed:

- Dependency versions reviewed

Result:

- Using updated versions (Spring Boot 3.x, Java 17)
- No known critical vulnerabilities identified

---

1.7 Authentication and Session Security

Tests performed:

- No token -> 401
- Expired token -> 401
- Wrong role -> 403

Implementation:

- JWT authentication with expiry
- Token validation on every request
- Password hashing using BCrypt

---

1.8 Data Integrity and Secrets Exposure

Risk checked:

- Accidental exposure of secrets

Implementation:

- .env added to .gitignore
- .env.example used for documentation
- Repository history checked

Result:

- No secrets found in commits

---

1.9 Logging and Monitoring

Implementation:

- Audit logs for create/update/delete operations
- Login attempts tracked
- Rate limit violations logged

Verification:

- Audit logs confirmed in database

---

1.10 Server-Side Request Forgery (SSRF)

Scenario tested:

- Sending internal URLs to AI endpoints

Implementation:

- Only text input accepted
- Suspicious inputs rejected

Result:

- Invalid requests return 400

---

2. Security Testing Performed

- Manual API testing using Postman
- SQL injection testing
- Prompt injection testing
- JWT authentication checks
- Rate limiting verification (curl scripts)
- OWASP ZAP baseline scan
- OWASP ZAP active scan
- Frontend XSS testing
- Git history audit for secrets

---

3. OWASP ZAP Results

Initial Findings:

- Missing security headers (medium severity)

Fix Applied:

- Added required headers in backend and AI service

Final Result:

- No Critical issues
- No High issues
- No Medium issues

---

4. Residual Risks

- HTTPS not configured in local Docker setup (development only)
- No account lockout mechanism (planned improvement)
- AI depends on external API (handled with fallback response)

---

5. Security Overview

Key protections implemented:

- JWT authentication
- Role-based access control
- Input validation and sanitisation
- Rate limiting
- Secure password storage (BCrypt)
- Audit logging
- Security headers

---

6. Final Notes

The focus was on implementing practical and testable security measures. Each major feature was tested with realistic misuse scenarios to ensure the system behaves correctly under unexpected inputs.

---

7. Team Sign-Off

All team members have reviewed and confirmed the implemented security measures.
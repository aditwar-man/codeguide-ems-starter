# Security Guidelines for codeguide-ems-starter (Enhanced EMS Project)

This document outlines security best practices and controls tailored to the `codeguide-ems-starter` repository as it evolves into the full-featured Educational Management System (EMS). It maps core security principles to the project’s architecture and technology stack.

---

## 1. Authentication & Access Control

- **Secure Password Storage**  
  • Use a strong hashing algorithm (Argon2 or bcrypt) with a unique salt per user.  
  • Enforce a robust password policy: minimum length (e.g., 12+ characters), complexity (uppercase, lowercase, digits, symbols), and periodic rotation for administrative accounts.

- **Session Management (Better Auth)**  
  • Generate cryptographically strong, unpredictable session identifiers.  
  • Store sessions in a server-side store (e.g., Redis) with idle and absolute timeouts.  
  • On logout or password change, invalidate all active sessions.  
  • Protect against session fixation by rotating the session ID after authentication.

- **Role-Based Access Control (RBAC)**  
  • Extend the `users` table with a `role` enum (`kurikulum`, `guru`, `siswa`).  
  • Implement Next.js `middleware.ts` to enforce route-group protection:  
    - Deny access to `/kurikulum/*` if `session.user.role !== 'kurikulum'`  
    - Deny `/guru/*` and `/siswa/*` similarly.  
  • Perform server-side role checks in every API route to prevent privilege escalation.

- **Multi-Factor Authentication (MFA)**  
  • Plan for optional MFA on sensitive accounts (e.g., Kurikulum admins).  
  • Integrate TOTP (Time-based One-Time Password) or SMS-based codes with backup codes.

---

## 2. Input Handling & Processing

- **Prevent Injection Attacks**  
  • Use Drizzle ORM’s parameterized queries exclusively to avoid SQL injection.  
  • Never concatenate user input into query strings or dynamic SQL.

- **Data Validation & Sanitization**  
  • Define Zod (or Joi) schemas for all API request bodies and query parameters.  
  • Enforce server-side validation in `/app/api` routes.  
  • For file uploads, validate MIME types, file extensions, and file size limits.

- **Cross-Site Scripting (XSS)**  
  • Sanitize user-generated content before rendering in React components.  
  • Use Next.js built-in `next/script` with `dangerouslySetInnerHTML` sparingly and only on trusted content.  
  • Deploy a strict Content Security Policy (CSP) header.

- **Prevent Template Injection**  
  • Avoid interpolating raw user input into templates.  
  • If dynamic templates are required, use a safe templating engine with built-in sanitization.

---

## 3. Data Protection & Privacy

- **Transport Encryption**  
  • Enforce HTTPS (TLS 1.2+). Redirect HTTP to HTTPS via `next.config.js` rewrites or load balancer settings.

- **Encryption at Rest**  
  • Rely on managed PostgreSQL encryption-at-rest features or configure AWS KMS/Azure Key Vault–backed disk encryption.

- **Secret Management**  
  • Store all secrets (DB credentials, API keys, JWT signing keys) in a secrets manager (Vault, AWS Secrets Manager, or environment variables loaded via Docker secrets).  
  • Avoid hard-coding secrets in the codebase or `.env` files committed to source control.

- **PII Handling**  
  • Mask or redact sensitive student and teacher information in logs and error messages.  
  • Implement data retention and deletion policies (e.g., GDPR right-to-be-forgotten).

---

## 4. API & Service Security

- **Rate Limiting & Throttling**  
  • Implement API rate limits (e.g., 100 req/min per IP) using middleware (e.g., `express-rate-limit` or a Next.js edge function with Redis).  
  • Protect authentication endpoints from brute-force attacks.

- **CORS Configuration**  
  • Restrict `Access-Control-Allow-Origin` to known front-end domains.  
  • Allow only necessary methods (`GET`, `POST`, `PUT`, `DELETE`).

- **API Versioning**  
  • Prefix routes with `/api/v1/` and plan for future `/v2/` changes to avoid breaking clients.

- **Principle of Least Privilege**  
  • Limit database user permissions: e.g., a read-only user for analytics endpoints.

---

## 5. Web Application Security Hygiene

- **Security Headers**  
  • `Content-Security-Policy`: Disallow inline scripts/styles, restrict `frame-ancestors`.  
  • `Strict-Transport-Security`: `max-age=63072000; includeSubDomains; preload`.  
  • `X-Content-Type-Options: nosniff`.  
  • `X-Frame-Options: DENY`.  
  • `Referrer-Policy: no-referrer-when-downgrade`.

- **CSRF Protection**  
  • Use NextAuth or custom CSRF tokens for state-changing requests.  
  • Validate tokens server-side on all POST/PUT/DELETE routes.

- **Secure Cookies**  
  • Set `HttpOnly`, `Secure`, and `SameSite=Strict` on session cookies.  
  • Avoid storing JWTs in `localStorage` or `sessionStorage`.

- **Subresource Integrity (SRI)**  
  • Add integrity attributes when loading third-party scripts or styles from CDNs.

---

## 6. File Upload & Real-Time Features

- **File Uploads (S3/GCS)**  
  • Use presigned URLs for direct client uploads.  
  • Validate file contents server-side (virus scanning, image mime-type checks).  
  • Store uploads outside the webroot with private ACLs.

- **Real-Time Chat (Socket.IO)**  
  • Host Socket.IO on a dedicated Node/Express server behind the Next.js app to maintain WebSocket persistence.  
  • Authenticate WebSocket connections by validating session tokens on handshake.  
  • Enforce per-user rate limits to prevent chat flooding.

---

## 7. Infrastructure & Configuration Management

- **Docker Security**  
  • Use minimal base images (e.g., `node:18-alpine`).  
  • Run containers as non-root users.  
  • Store secrets via Docker secrets or mounted volumes with restricted permissions.

- **CI/CD Pipeline**  
  • Integrate SCA and static-analysis tools in GitHub Actions (e.g., Dependabot for dependency updates, `npm audit`, `Snyk`).  
  • Run automated linting, testing, and container image scans on each push.

- **Server Hardening**  
  • Disable unused ports and services in production images.  
  • Keep system and library dependencies up to date.  
  • Remove debugging endpoints and verbose stack traces in production.

---

## 8. Dependency Management

- **Lockfiles**  
  • Commit `package-lock.json` to ensure repeatable builds.  
  • Audit dependencies regularly for vulnerabilities.

- **Minimize Attack Surface**  
  • Remove unused packages and disable optional features you do not require.  
  • Vet third-party components (e.g., shadcn/ui) and stay current with security patches.

---

## Conclusion
By embedding these security controls into the design, implementation, and deployment phases, the `codeguide-ems-starter` will evolve into a resilient, secure, and compliant Educational Management System. Regular reviews, automated scans, and adherence to the principle of least privilege will safeguard student and teacher data while maintaining high performance and developer agility.
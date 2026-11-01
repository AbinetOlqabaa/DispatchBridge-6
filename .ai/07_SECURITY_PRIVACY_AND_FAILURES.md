# Phase 7 — Security, Privacy, Validation and Failure Handling

## Threat model and isolation
Inspect actual auth, APIs, database rules, storage, file uploads, exports, share links, background jobs and secrets. Document trust boundaries and likely abuse paths. Deny by default. Enforce tenant/resource authorization server-side and in database rules where applicable; UI hiding is not security.

## Required controls
- Validate and normalize all untrusted input at server boundaries using the stack's schema-validation approach.
- Protect against IDOR, cross-tenant data leaks, privilege escalation, mass assignment, injection, unsafe file uploads, replay/duplicate submissions and race conditions.
- Use idempotency/transaction guards for consequential writes.
- Keep secrets only in supported secret management/server environment; provide `.env.example` placeholders only.
- Use secure session/cookie/token practices, expiry/revocation where supported, CSRF protection where relevant, rate limiting and abuse controls where supported.
- Limit upload types/size, verify ownership/access on download, and avoid public evidence URLs unless explicitly safe.
- Add security headers and restrictive CORS/Content Security Policy where compatible with deployment.
- Avoid sensitive values in logs/errors; return useful but non-revealing error messages.
- Apply pagination and bounded filters to list/export endpoints.
- Use UTC/server timestamps, explicit retention/deletion/backup behavior and minimal data collection.
- Offline data must be labeled unsynced and handle conflicts; do not promise unsupported offline capabilities.

### National Fayda e-ID & PII Privacy Controls
- **Restricted PII Classification:** FAN (16 digits) and FIN (12 digits) are classified as Restricted PII.
- **Server-Side Isolation:** Raw FAN and FIN values are stored strictly on the server-side memory/database model and are never returned in public or administrative API payloads.
- **Client Masking:** Only masked tokens (`maskedFan`: `•••• •••• •••• 3456`, `maskedFin`: `•••• •••• 9012`, `maskedPhone`: `+251 ••• ••3344`) are transmitted to the frontend.
- **Leading Zero Integrity:** Represented strictly as numeric strings (`/^\d{16}$/` and `/^\d{12}$/`) to preserve leading zeros without arithmetic distortion.

### OTP Lifecycle & Anti-Abuse
- **Purpose Binding:** Challenges are cryptographically partitioned by purpose (`REGISTRATION`, `PASSWORD_RESET`, `PROFILE_CHANGE`). A registration OTP cannot be submitted to reset a password.
- **Brute-Force Rate Limiting:** Challenges are capped at 5 failed verification attempts before permanent revocation.
- **Single-Use Semantics:** Upon successful verification, the challenge is immediately purged from memory to prevent replay attacks.
- **5-Minute Expiry:** All challenges automatically expire after 300 seconds.

### Identity Adapter Fail-Closed Policy
- **Production Mode:** `FaydaProductionAdapter` fails closed when official government API credentials are not provisioned, blocking unverified access.
- **Simulation Mode:** Clearly flagged via `faydaVerificationMethod: 'SIMULATION'` and prominent amber "Simulation / Dev Mode" indicators in the UI.

### Administrator Lockout & User Governance Protection
- **Lockout Prevention:** The server explicitly counts active users with the `SUPER_ADMIN` role prior to executing any suspension or role modification.
- **Boundary Rule:** If the target user is the sole remaining active Super Administrator (`activeSuperAdmins.length <= 1`), the server rejects the operation with HTTP 400 (`Cannot suspend or demote the last remaining active Super Administrator`).

### Profile Photo Upload Security
- **Size Constraint:** Hard 2MB payload ceiling enforced on client and server.
- **MIME Whitelisting:** Permitted image formats restricted to JPEG, PNG, and WebP data URIs. Executable and SVG vectors are rejected.

## Failure behavior
Every consequential action needs loading, success, failure, retry, and duplicate-submission behavior. Handle network loss, provider outage, expired session, stale form/version conflicts, missing configuration and permission denial honestly.

## Deliverables
Add/adjust threat-model notes and automated tests for security boundaries and failure cases. Record issues that require professional review or external setup. Never claim a compliance certification from these checks.

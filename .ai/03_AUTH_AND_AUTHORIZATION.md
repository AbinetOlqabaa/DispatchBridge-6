# Phase 3 — Authentication, Registration and Authorization

## Objective
Audit and complete authentication, Fayda e-ID national identity verification, mobile-first authentication, OTP lifecycle governance, and membership-scoped authorization without replacing working systems unnecessarily.

## Implemented Authentication & User Journeys

### 1. Mobile Number + Password Primary Authentication
- **Primary Credential:** Mobile phone number in E.164 format with country code selector (+251 Ethiopia default, +1 US, +254 Kenya, +253 Djibouti, +252 Somalia, +44 UK, +971 UAE, +256 Uganda, +250 Rwanda).
- **Fallback Credential:** Email address format seamlessly detected and supported.
- **Timing-Safe Generic Errors:** Uniform "Invalid mobile number, email, or password" errors to prevent account enumeration.
- **Suspension Enforcement:** Suspended accounts are immediately rejected with the audited suspension reason.

### 2. Public Registration & Role Restrictions
- **Allowed Public Roles:** Business Owner (`BUSINESS_OWNER`), Sender (`SENDER`), Receiver (`RECEIVER`).
- **Strict Super-Admin Restriction:** `SUPER_ADMIN` is removed from registration dropdowns and explicitly rejected server-side with HTTP 400 (`Public registration cannot create Super Administrator accounts`).
- **Super-Admin Bootstrap:** Controlled, one-time cryptographically random bootstrap token (`crypto.randomBytes(24).toString('hex')`) logged on startup. Redeemed via `/system/status` or `POST /api/auth/bootstrap-superadmin`. Once claimed, the token is permanently invalidated.

### 3. National Fayda e-ID Integration
- **FAN (Fayda Account Number):** Exactly 16 numeric digits. Leading zeros preserved using string type. Validated via `/^\d{16}$/`.
- **FIN (Fayda Identification Number):** Exactly 12 numeric digits. Leading zeros preserved. Validated via `/^\d{12}$/`.
- **Adapter Architecture:**
  - `FaydaSimulationAdapter`: Deterministic testing fixtures. Suffix `0000` simulates upstream gateway timeout. Suffix `999` or `9999` simulates credential mismatch. Prominent "Simulation / Dev Mode" indicator displayed in UI.
  - `FaydaProductionAdapter`: Fails closed when live government credentials are unconfigured.
- **Unique Atomic Linkage:** Mobile phone (E.164), FAN, and FIN are atomically linked to an internal opaque user ID (`usr_...`). Collisions on phone, FAN, or FIN are rejected.
- **Data Minimization & Masking:** Raw FAN and FIN are never exposed to clients or returned in API responses. They are masked at the server boundary (`•••• •••• •••• 3456`, `•••• •••• 9012`).

### 4. OTP Challenge Store & Lifecycle (`OtpAdapter`)
- **TTL:** 5-minute expiry window.
- **Rate Limiting:** 30-second resend cooldown (bypassed in automated test environment).
- **Brute-Force Protection:** Maximum 5 verification attempts per challenge before immediate invalidation.
- **Purpose Binding:** Challenges are strictly bound to their intended purpose (`REGISTRATION`, `PASSWORD_RESET`, `PROFILE_CHANGE`).
- **Single-Use Semantics:** OTP code is immediately consumed and deleted upon successful verification.
- **Simulation Mode:** Dev/simulation mode displays the generated OTP in an in-app banner and accepts test code `123456`.

### 5. Self-Service Recovery & Step-Up Security
- **Forgot Password:** Requires verified mobile number + FAN + FIN match. Server validates Fayda linkage and issues a bound `PASSWORD_RESET` OTP challenge to the user's mobile phone before allowing a password reset.
- **Signed-In Password Change:** Enforces step-up authentication. User must provide current password + verify a `PROFILE_CHANGE` OTP challenge sent to their bound mobile phone.

### 6. Administrator User Governance (`SuperAdminDashboard` & `/api/admin`)
- **Restricted Access:** Available exclusively to users with `SUPER_ADMIN` role.
- **Directory Search & Filters:** Search by name, phone, email, or business name; filter by role and status (`ACTIVE`, `SUSPENDED`).
- **Account Suspension & Reactivation:** Mandatory suspension justification reason; all state transitions logged to immutable audit trail.
- **Role Assignment & Lockout Protection:** Server-enforced protection preventing suspension or role demotion of the last remaining active Super Administrator (`Cannot suspend or demote the last remaining active Super Administrator`).

## Authorization Rules
- Resolve permissions from authenticated identity + active business membership + resource relationships.
- Deny by default; IDOR and cross-tenant boundaries enforced on all mutation and query endpoints.
- Raw FAN and FIN are stored server-side only; only masked tokens (`maskedFan`, `maskedFin`, `maskedPhone`) are exposed to client-side components.

# Phase 8 — Testing and Acceptance

## Rules
Inspect existing test tooling first. Reuse it. Never claim a command passed unless it was run in the current workspace. If a tool is unavailable, say so and give a manual alternative.

## Test layers
- Unit: validation, quantity arithmetic/decimal precision, state transitions, permission helpers, locale formatting, FAN/FIN regex formatting & masking.
- Integration: auth/session, membership authorization, dispatch creation, idempotency, view receipt, receipt submission, discrepancy review, notification outbox, and audit.
- Security: cross-tenant IDOR, unauthorized role actions, forged business IDs, mass assignment, expired/replayed invitation/share tokens, file access, export access, duplicate/replayed writes, public SUPER_ADMIN registration rejection, and super-admin lockout protection.
- UI/component: validation feedback, loading/empty/error/retry states, navigation, responsive tablet components, profile photo management, and profile summary cards.
- End-to-end: mobile registration/sign-in with Fayda verification & OTP challenge → onboarding/business selection → partner/location → dispatch with multiple lines → submit/share → recipient authenticated detail view → proof-of-view → receipt report → sender review/resolution → audit and notifications.
- Layer 6 (Identity, Fayda e-ID, OTP & Admin Governance):
  1. FAN and FIN numeric format validation & data masking (`•••• •••• •••• 3456`).
  2. Public self-registration strictly rejects `SUPER_ADMIN` role attempts.
  3. Fayda adapter simulation fixtures (deterministic timeouts and mismatches) & production fail-closed behavior.
  4. OTP challenge lifecycle (purpose binding, 5-attempt limits, 5-minute TTL, single-use invalidation).
  5. Mobile number authentication with E.164 normalization (+251 Ethiopia default).
  6. End-to-end registration flow with unique phone + FAN + FIN linkage & opaque `usr_...` ID.
  7. Forgot-password recovery with phone + FAN + FIN verification & OTP challenge.
  8. Signed-in password change with step-up mobile OTP challenge.
  9. Profile photo upload size limits (<= 2MB), MIME format validation, and photo removal.
  10. Admin user governance (search, filter, suspend, reactivate) & lockout protection for the last remaining active super-admin.
- Regression: all previously working features, including map/telemetry, scanner/checkpoints, PDF manifest, CSV import/export, analytics, languages and theme.

## Responsive test matrix
At minimum check 390x844, 768x1024, 820x1180, 1024x768 and 1440x900. Prefer also 360x800, 1180x820 and 1280x800. Test every major route for both tablet orientations. Record routes, viewports, methods and results.

## Release gates
- Lint/typecheck/build passes (`npm run lint` -> 0 errors, `compile_applet` -> PASS).
- Relevant tests pass: **38/38 automated tests passing with 0 failures** (`npm test`).
- No known critical security issue left unacknowledged.
- All required flows work with real configured persistence, or sandbox-only limitations are clearly documented.
- No false provider delivery claims.
- Tablet layouts pass the viewport matrix across portrait (768x1024, 820x1180) and landscape (1024x768, 1180x820).
- No critical flow is implemented as a fake success.

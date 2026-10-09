# Design

## Context

The backend (Spring Boot 4, H2) exposes an unauthenticated REST API under `/api/ads`; the React SPA runs on a different origin (`http://localhost:5173`). See proposal.md for motivation. The `ads-api` spec currently allows CORS for `GET`/`POST`/`PUT` without credentials. Requirements are in `specs/user-auth` and `specs/ads-api`.

## Goals / Non-Goals

**Goals:**
- A cookie-session login that works across the SPA/API origins in dev and prod.
- Safe-by-default handling: hashed passwords, CSRF protection, no user enumeration on login.
- A contract both repos can implement and test independently.

**Non-Goals:**
- Password reset, email verification, roles/admin, account deletion, rate limiting, "remember me", social login.
- Delete-ad endpoint (still not part of the API).

## Decisions

**Server session cookie, not JWT.** Chosen by the user. The session id lives in an `HttpOnly`, `SameSite=Lax` cookie; the backend stores session state server-side (in memory is sufficient for the H2-backed single instance). Alternative: JWT bearer, rejected because logout/revocation and storage in the SPA are harder.

**CSRF via double-submit token cookie.** The backend issues a readable `XSRF-TOKEN` cookie; the SPA echoes it in `X-XSRF-TOKEN` on `POST`/`PUT`. This is Spring Security's cookie CSRF repository convention and Axios/fetch-friendly. Login and register are exempt because no session exists yet. `SameSite=Lax` alone is not relied upon since the API is cross-origin in dev.

**Credentialed CORS.** Because cookies are sent, `Access-Control-Allow-Origin` must be the explicit configured origin and `Access-Control-Allow-Credentials: true`. The SPA uses `credentials: "include"` on every request. Both dev origins (`localhost:5173` and `localhost:8080`) are same-site (localhost), so `SameSite=Lax` cookies are sent.

**Ownership by username on the ad.** The ad stores an owner reference; the API exposes it as `author` (username). Usernames are immutable in this change, so exposing the username is stable. Alternative: exposing the user `id`; rejected as less useful for display and the SPA already has the username from `/api/auth/me`.

**Legacy ads are ownerless and read-only.** Existing rows get a null owner; no one can edit them. Alternative: let any logged-in user claim them, rejected as an unintended permission grant.

**Status codes.** `401` for no/invalid session, `403` for authenticated-but-forbidden (non-owner, missing CSRF token), `409` for a taken username. All errors remain RFC 9457 problem details, consistent with `ads-api`.

**Password handling.** Backend stores a salted adaptive hash (bcrypt-class); the 72-character cap matches bcrypt's input limit. Login failure messages are identical for unknown user and wrong password.

## Risks / Trade-offs

- [In-memory sessions are lost on restart and don't scale horizontally] → Acceptable for this app; revisit with a shared session store if scaled out.
- [Cookie auth requires correct CORS/CSRF wiring in both repos] → Contract scenarios for preflight and missing token; integration check in the final task group.
- [Existing ads become uneditable] → Documented behavior; seed data or future "claim" feature can address.
- [Production with different registrable domains would break `SameSite=Lax` for cross-site `fetch`] → Serve SPA and API from the same site, or revisit cookie settings; out of scope here.
- [Username enumeration through registration `409`] → Accepted trade-off for self-registration.

## Clarifications

Contract gaps found during implementation. Add a line here (by pull request) when a repo has to decide something the specs leave open, so every repo makes the same call.

- Backend: CSRF is enforced only when the request carries a session. An anonymous `POST`/`PUT` gets `401`, not a CSRF `403`.

# Design

## Context

See proposal.md for motivation. Current state that shapes the approach:

- **Backend** (`bulletin-board-backend`): Spring Boot 4.1.1, Java 25, Spring MVC + Data JPA + Validation, in-memory H2 with `ddl-auto=create-drop`. Feature packages under `com.example.bulletinboard` (`ad/`, `config/`). Conventions: records for DTOs, constructor injection, no Lombok, errors as RFC 9457 `ProblemDetail` (`spring.mvc.problemdetails.enabled=true`, `AdNotFoundException extends ErrorResponseException`). No Spring Security. `config/CorsConfig` allows only GET/POST/PUT from `app.cors.allowed-origins` with no credentials.
- **Frontend** (`bulletin-board-frontend`): React 19 + Vite 8 + TS 6, single page in `App.tsx`, no router, no state library, no test tooling. All HTTP goes through `request<T>()` in `src/api.ts`, which already turns problem-details into `Error(detail)`. Vite proxies `/api` to `localhost:8080`, so in development the browser sees one origin.
- Tests: backend uses `@SpringBootTest` + `@AutoConfigureMockMvc` (`AdControllerTests` style). Frontend has none.

## Goals / Non-Goals

**Goals:**
- Server-side session auth that works same-origin via the Vite proxy and cross-origin via `VITE_API_BASE_URL`.
- Keep `/api/ads` request/response behavior byte-for-byte unchanged for existing clients.
- Follow existing backend conventions (feature package, records, ProblemDetail).

**Non-Goals:**
- Ad ownership, restricting ad create/edit to signed-in users (follow-up change).
- Email verification, password reset, change password, account deletion, "remember me".
- Rate limiting / account lockout on failed sign-in.
- Persistent database or migrations (Flyway etc.).
- Client-side routing library.

## Decisions

### D1. Session cookie via Spring Security, not JWT
Use `spring-boot-starter-security` with a stateful `HttpSession` (`JSESSIONID`, HttpOnly). Sign-in is a JSON endpoint in our own controller that calls `AuthenticationManager` and stores the `SecurityContext` via `HttpSessionSecurityContextRepository`; form login and HTTP Basic are disabled.
- *Why*: no extra library, HttpOnly cookie keeps credentials out of JS, logout is a real server-side invalidation, and the Vite proxy makes it same-origin in dev.
- *Alternatives*: JWT in `Authorization` header (needs a JWT lib, token storage in JS, no server-side revocation); Spring `formLogin()` (redirect/HTML-oriented responses don't fit a JSON API).

### D2. API surface (`/api/auth`)
| Method | Path | Body | Success | Errors |
|---|---|---|---|---|
| POST | `/api/auth/register` | `{email, password}` | 201 `{id, email}`, session created | 400, 409 |
| POST | `/api/auth/login` | `{email, password}` | 200 `{id, email}` | 400, 401 |
| POST | `/api/auth/logout` | – | 204 | – |
| GET | `/api/auth/me` | – | 200 `{id, email}` | 401 |

Session id and CSRF token are rotated on login/register (Spring Security's `ChangeSessionIdAuthenticationStrategy` + `CsrfAuthenticationStrategy`, applied explicitly since we authenticate manually; the new token is loaded so the same response sets the new `XSRF-TOKEN` cookie). The cookie `CsrfTokenRepository` is a shared bean used by both the filter chain and `AuthController`. Unauthenticated access to `/api/auth/me` returns 401 ProblemDetail via a custom `AuthenticationEntryPoint` (no `WWW-Authenticate: Basic` prompt). `/api/ads/**`, `/api/auth/register`, `/api/auth/login`, `/api/auth/logout`, and `/h2-console/**` are `permitAll`.

### D3. CSRF: Spring Security SPA mode, scoped to `/api/auth/**`
Enable CSRF with the cookie token repository (`XSRF-TOKEN` cookie, `X-XSRF-TOKEN` header — `csrf.spa()` in Spring Security 7) and ignore `/api/ads/**` and `/h2-console/**`.
- *Why*: once a session cookie exists, state-changing auth calls need CSRF protection; ad endpoints are anonymous today, so CSRF adds nothing there and enforcing it would break existing API clients (spec: "Ads remain available anonymously"). The ownership change will move `/api/ads` under CSRF.
- The frontend reads `XSRF-TOKEN` from `document.cookie` and sends it as `X-XSRF-TOKEN` on non-GET requests; `GET /api/auth/me` on app load ensures the cookie is issued.
- *Alternative*: disable CSRF and rely on `SameSite=Lax` — weaker and depends on browser defaults.

### D4. User model
New package `com.example.bulletinboard.user`: `User` entity (table `users`; `id` IDENTITY, `email` unique not-null, `passwordHash`, `createdAt`), `UserRepository` (`findByEmail`), `UserService` (register, normalizes email with `trim().toLowerCase(Locale.ROOT)`), `UserDetailsService` adapter, `AuthController`, records `RegisterRequest`/`LoginRequest` (`@NotBlank @Email @Size(max=254)`, password `@Size(min=8)` plus a custom `@MaxUtf8Bytes(72)` on registration) and `UserResponse.from(User)`. Duplicate email → `EmailAlreadyRegisteredException extends ErrorResponseException` (409). Unique DB constraint backs the service check to cover races (`DataIntegrityViolationException` → 409).
- Password hashing: `PasswordEncoderFactories.createDelegatingPasswordEncoder()` (bcrypt default). bcrypt rejects input over 72 *bytes* (Spring Security throws `IllegalArgumentException`), so the limit is checked on UTF-8 bytes, not characters — a character limit lets multibyte passwords through and turns them into a 500.
- Failed sign-in: catch `AuthenticationException` → 401 ProblemDetail with detail "Invalid email or password" for both unknown email and wrong password.

### D5. CORS for credentials
Extend `CorsConfig` (methods stay GET/POST/PUT — logout is a POST): set `allowCredentials(true)`, `allowedHeaders("Content-Type", "X-XSRF-TOKEN")`, and keep explicit origins (wildcards are not allowed with credentials). Register CORS with Spring Security (`http.cors(...)`) so preflights are not blocked by the filter chain. Frontend `request()` uses `credentials: 'include'`.

### D6. Frontend structure without a router
- `src/api.ts`: add `credentials: 'include'`, CSRF header injection, and `register`, `login`, `logout`, `getCurrentUser` (`getCurrentUser` maps 401 and 404 to `null`).
- `src/auth.tsx`: `AuthProvider` + `useAuth()` context holding `user | null | 'loading'`, calls `getCurrentUser` on mount.
- `src/AuthForm.tsx`: one controlled form with `mode: 'login' | 'register'`, mirroring `AdForm`'s error/saving handling.
- `App.tsx`: header showing email + Sign out when signed in, or Sign in / Register toggles that reveal `AuthForm`. Ads section unchanged.
- *Alternative*: add react-router with `/login` and `/register` routes — more dependency and restructuring than two forms justify; revisit when more pages exist.

### D7. Frontend test tooling
Add Vitest + `@testing-library/react` + `@testing-library/user-event` + `jsdom` with an `npm test` script, mocking `fetch`. Needed to verify the auth UI requirements; reuses Vite config.

## Risks / Trade-offs

- [In-memory H2 with `create-drop` loses all accounts and sessions on restart] → Accepted for this stage; documented in backend README. A persistence change is separate.
- [In-memory `HttpSession` doesn't scale across instances] → Single instance today; Spring Session can be added later without API changes.
- [No brute-force protection on `/api/auth/login`] → Out of scope; note as follow-up. bcrypt cost slows offline attacks only.
- [Cross-origin deployment requires `SameSite=None; Secure` cookies] → Dev uses the proxy (same-origin). Document that cross-origin builds need HTTPS and cookie config; default stays `Lax`.
- [CSRF ignored on `/api/ads/**`] → No authenticated state is attached to ad operations yet, so there is nothing to forge; revisit in the ownership change.
- [Adding Spring Security changes defaults for every endpoint (e.g. security headers, `/h2-console` frames)] → Explicit `permitAll` rules, `frameOptions().sameOrigin()` for the H2 console, and the existing `AdControllerTests` (including the CORS preflight test) must pass unchanged.

## Migration Plan

Additive; no data migration (schema is recreated on startup). Deploy backend first — the old frontend keeps working because `/api/ads` is unchanged — then the frontend. Rollback: revert either repo independently; the frontend handles a 404 from `/api/auth/me` by treating the user as signed out.

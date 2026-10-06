# Tasks

Repos: **BE** = `bulletin-board-backend`, **FE** = `bulletin-board-frontend`. Backend groups (1–3) come first; the frontend (4–6) depends on the `/api/auth` contract in design.md D2.

## 1. BE: Security foundation

- [x] 1.1 Add `spring-boot-starter-security` and `spring-boot-starter-security-test` (test scope) to `pom.xml`; verify `./mvnw dependency:tree` lists both
- [x] 1.2 Add `config/SecurityConfig` per design D1–D3/D5: stateful sessions, form login and HTTP Basic disabled, `permitAll` for `/api/ads/**`, `/api/auth/register|login|logout`, `/h2-console/**`; authenticated `/api/auth/me`; JSON 401 `AuthenticationEntryPoint`; SPA CSRF ignoring `/api/ads/**` and `/h2-console/**`; H2 console frame options `sameOrigin`; `http.cors(...)`; delegating `PasswordEncoder` bean; verify existing `AdControllerTests` and `contextLoads` pass unchanged with `./mvnw test`
- [x] 1.3 Update `CorsConfig` with `allowCredentials(true)` and allowed headers `Content-Type`, `X-XSRF-TOKEN`; extend `AdControllerTests.allowsFrontendOrigin` (or add a sibling test) asserting `Access-Control-Allow-Credentials: true` on a preflight, and verify it passes

## 2. BE: User accounts and registration

- [x] 2.1 Create `user/User` entity (`users` table, unique `email`, `passwordHash`, `createdAt`) and `UserRepository.findByEmail`; verify with a `@DataJpaTest` that saving two users with the same email throws `DataIntegrityViolationException`
- [x] 2.2 Create records `RegisterRequest`, `LoginRequest`, `UserResponse.from(User)` and `EmailAlreadyRegisteredException` (409 ProblemDetail); verify `UserResponse` has no password field (covered by 2.4 JSON assertions)
- [x] 2.3 Implement `UserService.register` (normalize email `trim().toLowerCase(Locale.ROOT)`, hash password, map unique-constraint violation to 409) and a `UserDetailsService` adapter; verify with unit/integration tests for normalization and duplicate detection
- [x] 2.4 Add `AuthController` `POST /api/auth/register` that creates the user, authenticates, rotates the session, and saves the `SecurityContext`; verify `AuthControllerTests` (MockMvc, `@BeforeEach` deleteAll, `with(csrf())`) cover: 201 with `{id,email}` and no password fields, session is authenticated afterwards, 400 for invalid email / 7-char / 73-char password, 409 for ` Alice@Example.COM ` after `alice@example.com`

## 3. BE: Sign-in, sign-out, current user

- [x] 3.1 Add `POST /api/auth/login` (authenticate via `AuthenticationManager`, rotate session, save context; `AuthenticationException` → 401 "Invalid email or password"); verify tests: 200 with case-insensitive email, 401 with identical detail for wrong password and unknown email, 400 for blank fields
- [x] 3.2 Add `GET /api/auth/me` and `POST /api/auth/logout` (invalidate session, clear context, 204 even when anonymous); verify tests: `me` 200 when signed in, 401 ProblemDetail when anonymous (no `WWW-Authenticate: Basic`), 401 after logout, logout 204 when anonymous, and a POST to `/api/auth/login` without a CSRF token is rejected with 403
- [x] 3.3 Add a test that anonymous `POST /api/ads` without a CSRF token still returns 201 (spec "Ads remain available anonymously"); verify it passes
- [x] 3.4 Document the `/api/auth` endpoints, CSRF header, and "accounts are lost on restart (in-memory H2)" note in backend `README.md`; verify the README API table matches design D2
- [x] 3.5 Review fix: limit registration passwords to 72 UTF-8 bytes (`@MaxUtf8Bytes`) instead of 72 characters; verify `AuthControllerTests` reject an 84-byte Cyrillic password with 400 and accept a 72-byte one
- [x] 3.6 Review fix: rotate the CSRF token on sign-in/registration via `CsrfAuthenticationStrategy` with a shared `CsrfTokenRepository` bean; verify `CsrfCookieTests` sees a new `XSRF-TOKEN` in the register response and can log out with it

## 4. FE: Test tooling

- [x] 4.1 Add dev deps `vitest`, `jsdom`, `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`; configure `test` in `vite.config.ts` (jsdom, setup file) and add `"test": "vitest run"`; verify `npm test` runs a trivial smoke test and `npm run build` still succeeds

## 5. FE: Auth API and state

- [x] 5.1 Update `request()` in `src/api.ts` to send `credentials: 'include'` and, for non-GET requests, an `X-XSRF-TOKEN` header read from the `XSRF-TOKEN` cookie; verify a unit test with mocked `fetch` asserts both for `createAd` and that existing ad calls are otherwise unchanged
- [x] 5.2 Add `User` type and `register`, `login`, `logout`, `getCurrentUser` (401/404 → `null`) to `src/api.ts`; verify unit tests cover success, 401-to-null, and problem-detail error messages
- [x] 5.3 Add `src/auth.tsx` with `AuthProvider`/`useAuth()` (loads current user on mount; exposes `login`, `register`, `logout`); wrap `<App/>` in `main.tsx`; verify a test that a mocked signed-in `me` response yields the user after mount (reload persistence)

## 6. FE: Sign-in and registration UI

- [x] 6.1 Add `src/AuthForm.tsx` (`mode: 'login' | 'register'`, email + password inputs, saving/error state like `AdForm`); verify tests: submitting calls the right API, server detail "Invalid email or password" is displayed on failure
- [x] 6.2 Add an auth header to `App.tsx`: when signed in show email + Sign out; when signed out show Sign in / Register toggles revealing `AuthForm`; ads section unchanged; add styles to `index.css`; verify tests for register → shows email, sign out → shows Sign in, and `npm run lint` + `npm run build` pass
- [x] 6.3 Update frontend `README.md` noting cookie-based sign-in and that cross-origin builds (`VITE_API_BASE_URL`) need HTTPS and matching backend CORS origin; verify the README run instructions still work as written

## 7. Integration check

- [x] 7.1 Run backend (`./mvnw spring-boot:run`) and frontend (`npm run dev`) together and manually verify: register, reload stays signed in, sign out, sign in with wrong then right password, and create/edit an ad while signed out and while signed in

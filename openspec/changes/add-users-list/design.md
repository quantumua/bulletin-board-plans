# Design

## Context

Accounts live in the `users` table (`AppUser`: `id`, `username`, `usernameLower`, `passwordHash`) with no timestamp, so "registration date" needs new data. Auth is a cookie session with CSRF; unauthenticated API calls return `401` problem details. The frontend is a single view (ads) with an `AuthBar`; there is no navigation yet.

## Goals / Non-Goals

**Goals:**
- Logged-in users can open a separate "Users" tab listing username and registration date.
- A minimal, privacy-conscious response shape (no hashes, no lowercase key).

**Non-Goals:**
- Pagination, search, or sorting options (the board is small; revisit if the list grows).
- Profiles, per-user ad lists, roles, or deleting users.
- Changing the register/login/me response shapes.

## Decisions

- **New capability `user-directory`** rather than extending `user-auth`: listing users is a read concern with its own access rules, and keeps `user-auth` focused on sessions.
- **`GET /api/users` requires login** (user's choice) instead of being public; anonymous gets `401`, consistent with `/api/auth/me`.
- **`createdAt` stored on the user row**, set at registration, UTC ISO-8601 like ads. Data is in-memory H2 and recreated on start, so no backfill is needed; the backend plan should still make the column non-null with a default for safety.
- **Dedicated response type** with `id`, `username`, `createdAt` only, so entity fields can never leak by accident.
- **Tab is frontend-only state** (no router required); the tab is hidden when anonymous, and if the session expires the `401` sends the user back to the ads view.

## Risks / Trade-offs

- Any logged-in user sees every username: acceptable for now, but usernames are login identifiers, which eases targeted guessing. Rate limiting on login is out of scope here.
- No pagination: response grows with users; acceptable at current scale.

## Clarifications

Contract gaps found during implementation (backend). Add a line here when a repo has to decide something the specs leave open.

- A logged-in `POST`/`PUT`/`DELETE` to `/api/users` returns `405` only when it carries a valid CSRF token. Without the token the CSRF check runs first and the response is `403`. Anonymous requests are `401` either way.
- Anonymous `401` responses start no session on any endpoint (the request cache is disabled), so no `JSESSIONID` is set and an anonymous write after an anonymous read stays `401` instead of becoming a CSRF `403`.
- "Sets no cookies" for `GET /api/users` includes the `XSRF-TOKEN` cookie that other endpoints issue; the backend suppresses it on this path, for logged-in and anonymous callers alike.

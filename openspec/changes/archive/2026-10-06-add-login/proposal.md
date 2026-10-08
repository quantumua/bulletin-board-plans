# Proposal

## Why

The bulletin board has no concept of a user: every visitor is anonymous and nothing ties an ad to the person who posted it. Before features like ad ownership or "my ads" can exist, people need a way to create an account and sign in. This change adds email + password registration and login as that foundation.

## What Changes

- Visitors can register an account with an email address and a password; registering signs them in.
- Registered users can sign in with email + password and sign out.
- The application can tell a client who the signed-in user is (or that nobody is signed in), and the sign-in survives a page reload.
- Passwords are stored only as salted one-way hashes and are never returned by the API.
- The frontend gets Sign in and Register screens, and shows the signed-in user's email with a Sign out action.
- Existing ad browsing, creation and editing keep working anonymously exactly as today — this change does **not** restrict any ad endpoint or add ad ownership (planned as a follow-up change).

## Capabilities

### New Capabilities
- `user-auth`: User accounts identified by email, password-based registration, sign-in, sign-out, and discovery of the current signed-in user.

### Modified Capabilities
<!-- None: there are no existing specs in openspec/specs/, and ad behavior is unchanged. -->

## Impact

- **bulletin-board-backend** (Spring Boot 4.1, Java 25): adds Spring Security, a `user` feature package (entity, repository, service, controller), new `/api/auth/**` endpoints, security configuration, and CORS changes to allow credentials. New dependencies: `spring-boot-starter-security` and its test starter.
- **bulletin-board-frontend** (React 19, Vite): `api.ts` sends credentials and the CSRF header; new auth API functions, auth state, sign-in/register forms, and a header with the signed-in user. Adds Vitest + React Testing Library as dev dependencies (the project has no test tooling today).
- **Data**: new `users` table. The backend still uses in-memory H2 with `create-drop`, so accounts are lost on restart — acceptable for now, called out in design.
- **APIs**: additive only; no breaking changes to `/api/ads`.

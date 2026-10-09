# Spec Delta

## Purpose

Defines how users of the bulletin board register, log in and out, and how the frontend learns who is logged in, using a cookie-based server session.

## ADDED Requirements

### Requirement: User registration
The API SHALL create an account from `POST /api/auth/register` with a JSON body containing `username` and `password`. A `username` MUST be 3 to 50 characters and unique ignoring case; a `password` MUST be 8 to 72 characters. The password SHALL NOT be returned by any endpoint.

#### Scenario: Valid registration
- **WHEN** a client posts a unique valid `username` and a valid `password`
- **THEN** the response is `201 Created` with a JSON body containing `id` and `username`, and no session is established

#### Scenario: Username already taken
- **WHEN** a client registers a `username` that already exists, ignoring case
- **THEN** the response is `409 Conflict` with an RFC 9457 problem detail and no account is created

#### Scenario: Invalid credentials format
- **WHEN** a client registers with a `username` or `password` outside the allowed lengths
- **THEN** the response is `400 Bad Request` with an RFC 9457 problem detail and no account is created

### Requirement: Login
The API SHALL authenticate a user from `POST /api/auth/login` with a JSON body containing `username` and `password`, and on success establish a session through a `Set-Cookie` header whose cookie is `HttpOnly` and `SameSite=Lax`.

#### Scenario: Valid credentials
- **WHEN** a client posts a correct `username` and `password`
- **THEN** the response is `200 OK` with a JSON body containing `id` and `username`, and sets the session cookie

#### Scenario: Invalid credentials
- **WHEN** a client posts an unknown `username` or a wrong `password`
- **THEN** the response is `401 Unauthorized` with an RFC 9457 problem detail that does not reveal which of the two was wrong, and no session cookie is set

### Requirement: Current user
The API SHALL return the logged-in user from `GET /api/auth/me`.

#### Scenario: Logged in
- **WHEN** a client calls `GET /api/auth/me` with a valid session cookie
- **THEN** the response is `200 OK` with a JSON body containing `id` and `username`

#### Scenario: Not logged in
- **WHEN** a client calls `GET /api/auth/me` without a valid session
- **THEN** the response is `401 Unauthorized` with an RFC 9457 problem detail

### Requirement: Logout
The API SHALL end the current session from `POST /api/auth/logout` and expire the session cookie.

#### Scenario: Logout while logged in
- **WHEN** a logged-in client posts to `/api/auth/logout`
- **THEN** the response is `204 No Content`, the session cookie is expired, and later calls with that cookie are treated as anonymous

#### Scenario: Logout while anonymous
- **WHEN** a client without a session posts to `/api/auth/logout`
- **THEN** the response is `204 No Content`

### Requirement: CSRF protection
The API SHALL reject every state-changing request (`POST`, `PUT`) to `/api/**` that carries a session cookie but lacks a valid `X-XSRF-TOKEN` header, and SHALL deliver the token to the frontend in a readable `XSRF-TOKEN` cookie. The register and login requests are exempt from the header requirement.

#### Scenario: Missing token
- **WHEN** a logged-in client posts or puts to `/api/ads` without the `X-XSRF-TOKEN` header
- **THEN** the response is `403 Forbidden` with an RFC 9457 problem detail and nothing is changed

#### Scenario: Valid token
- **WHEN** a logged-in client sends the `XSRF-TOKEN` cookie value in the `X-XSRF-TOKEN` header
- **THEN** the request is processed normally

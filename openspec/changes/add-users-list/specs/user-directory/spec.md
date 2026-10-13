## ADDED Requirements

### Requirement: User representation
The API SHALL represent a listed user as a JSON object with exactly the fields `id` (integer), `username` (string, original casing) and `createdAt` (ISO-8601 UTC timestamp of registration). No endpoint SHALL return a password or password hash.

#### Scenario: User returned by the directory
- **WHEN** `GET /api/users` returns a user
- **THEN** the object contains `id`, `username` and `createdAt` and no other fields

### Requirement: List registered users
The API SHALL return all registered users from `GET /api/users`, ordered newest first by `createdAt`, to logged-in clients only. The endpoint SHALL set no cookies and clear none.

#### Scenario: Logged in with users registered
- **WHEN** a logged-in client calls `GET /api/users` and two users exist
- **THEN** the response is `200 OK` with a JSON array of both users, the most recently registered first

#### Scenario: Only the caller exists
- **WHEN** the only registered user calls `GET /api/users` while logged in
- **THEN** the response is `200 OK` with a JSON array containing that single user

#### Scenario: Anonymous
- **WHEN** a client without a valid session calls `GET /api/users`
- **THEN** the response is `401 Unauthorized` with an RFC 9457 problem detail and no user data

#### Scenario: Error precedence
- **WHEN** a client calls `GET /api/users`
- **THEN** authentication is checked first, so an anonymous request is always `401`; a logged-in `GET` has no body or parameters and cannot produce `400`, `403`, `404` or `409`

#### Scenario: Write methods not offered
- **WHEN** a client sends `POST`, `PUT` or `DELETE` to `/api/users`
- **THEN** no user is created, changed or removed (`405 Method Not Allowed` when logged in, `401` when anonymous)

### Requirement: Registration time recorded
The system SHALL record the time of registration as the user's `createdAt` when an account is created via `POST /api/auth/register`. The register response body defined by `user-auth` is unchanged.

#### Scenario: New account appears with its time
- **WHEN** a client registers successfully and a logged-in client then calls `GET /api/users`
- **THEN** the new user is listed with a `createdAt` at or after the moment of registration

### Requirement: Paths outside the API
Adding the login requirement for `/api/users` SHALL NOT change the behavior of other paths: public reads under `/api/ads` stay anonymous, and non-API paths such as the H2 console keep answering `401`.

#### Scenario: Ads remain public
- **WHEN** an anonymous client calls `GET /api/ads`
- **THEN** the response is `200 OK`

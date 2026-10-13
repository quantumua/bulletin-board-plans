# Spec Delta

## ADDED Requirements

### Requirement: Authentication for writes
The API SHALL require a logged-in user (see `user-auth`) for `POST /api/ads` and `PUT /api/ads/{id}`, while `GET /api/ads` and `GET /api/ads/{id}` remain available to anonymous clients.

#### Scenario: Anonymous create
- **WHEN** an anonymous client posts to `/api/ads`
- **THEN** the response is `401 Unauthorized` with an RFC 9457 problem detail and no ad is created

#### Scenario: Anonymous read
- **WHEN** an anonymous client calls `GET /api/ads`
- **THEN** the response is `200 OK` as for any other client

### Requirement: Ad ownership
The API SHALL record the logged-in user who creates an ad as its owner and SHALL allow only the owner to edit it. Ads that existed before login was introduced have no owner and SHALL NOT be editable.

#### Scenario: Owner edits
- **WHEN** the owner puts a valid update for their ad
- **THEN** the response is `200 OK` with the updated ad

#### Scenario: Non-owner edits
- **WHEN** a logged-in user puts an update for an ad owned by someone else
- **THEN** the response is `403 Forbidden` with an RFC 9457 problem detail and the ad is unchanged

#### Scenario: Ownerless ad
- **WHEN** a logged-in user puts an update for an ad that has no owner
- **THEN** the response is `403 Forbidden` with an RFC 9457 problem detail

## MODIFIED Requirements

### Requirement: Ad representation
The API SHALL represent an ad as a JSON object with the fields `id` (integer), `title` (string), `description` (string), `author` (string username of the owner, or `null` for ownerless ads), `createdAt` and `updatedAt` (ISO-8601 UTC timestamps).

#### Scenario: Ad returned by the API
- **WHEN** any endpoint returns an ad
- **THEN** the response contains `id`, `title`, `description`, `author`, `createdAt` and `updatedAt`

### Requirement: Create an ad
The API SHALL create an ad from `POST /api/ads` with a JSON body containing `title` and `description`, owned by the logged-in user. Any `author` supplied in the body SHALL be ignored.

#### Scenario: Valid ad
- **WHEN** a logged-in client posts a valid `title` and `description`
- **THEN** the response is `201 Created` with the new ad whose `author` is the logged-in username, and a `Location` header pointing to `/api/ads/{id}`

### Requirement: Edit an ad
The API SHALL replace the `title` and `description` of an existing ad from `PUT /api/ads/{id}` and refresh its `updatedAt`. The `author` SHALL NOT change.

#### Scenario: Valid update
- **WHEN** the owner puts a valid `title` and `description` for their existing ad
- **THEN** the response is `200 OK` with the updated ad, the same `id`, `author` and `createdAt`, and a new `updatedAt`

#### Scenario: Ad does not exist
- **WHEN** a logged-in client puts an update for an unknown id
- **THEN** the response is `404 Not Found` with an RFC 9457 problem detail

### Requirement: Cross-origin access
The API SHALL allow `GET`, `POST` and `PUT` requests to `/api/**` from configured frontend origins, by default `http://localhost:5173`, including credentialed requests (cookies) and the `Content-Type` and `X-XSRF-TOKEN` request headers. The allowed origin SHALL be echoed explicitly, never `*`.

#### Scenario: Frontend preflight
- **WHEN** the browser sends a CORS preflight from `http://localhost:5173`
- **THEN** the response allows that origin, allows credentials, and allows the `Content-Type` and `X-XSRF-TOKEN` headers

#### Scenario: Credentialed request from unknown origin
- **WHEN** a request arrives from an origin that is not configured
- **THEN** the response does not allow that origin

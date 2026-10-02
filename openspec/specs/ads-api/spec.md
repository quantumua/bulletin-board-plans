# ads-api Specification

## Purpose
HTTP contract between the bulletin board backend and frontend for listing, creating, viewing and editing ads.

## Requirements

### Requirement: Ad representation
The API SHALL represent an ad as a JSON object with the fields `id` (integer), `title` (string), `description` (string), `createdAt` and `updatedAt` (ISO-8601 UTC timestamps).

#### Scenario: Ad returned by the API
- **WHEN** any endpoint returns an ad
- **THEN** the response contains `id`, `title`, `description`, `createdAt` and `updatedAt`

### Requirement: List ads
The API SHALL return all ads from `GET /api/ads`, ordered newest first by `createdAt`.

#### Scenario: Ads exist
- **WHEN** a client calls `GET /api/ads` and two ads exist
- **THEN** the response is `200 OK` with a JSON array of both ads, the most recently created first

#### Scenario: No ads
- **WHEN** a client calls `GET /api/ads` and no ads exist
- **THEN** the response is `200 OK` with an empty JSON array

### Requirement: View an ad
The API SHALL return a single ad from `GET /api/ads/{id}`.

#### Scenario: Ad exists
- **WHEN** a client calls `GET /api/ads/{id}` for an existing ad
- **THEN** the response is `200 OK` with that ad

#### Scenario: Ad does not exist
- **WHEN** a client calls `GET /api/ads/{id}` for an unknown id
- **THEN** the response is `404 Not Found` with an RFC 9457 problem detail whose `detail` names the missing ad

### Requirement: Create an ad
The API SHALL create an ad from `POST /api/ads` with a JSON body containing `title` and `description`.

#### Scenario: Valid ad
- **WHEN** a client posts a valid `title` and `description`
- **THEN** the response is `201 Created` with the new ad and a `Location` header pointing to `/api/ads/{id}`

### Requirement: Edit an ad
The API SHALL replace the `title` and `description` of an existing ad from `PUT /api/ads/{id}` and refresh its `updatedAt`.

#### Scenario: Valid update
- **WHEN** a client puts a valid `title` and `description` for an existing ad
- **THEN** the response is `200 OK` with the updated ad, the same `id` and `createdAt`, and a new `updatedAt`

#### Scenario: Ad does not exist
- **WHEN** a client puts an update for an unknown id
- **THEN** the response is `404 Not Found` with an RFC 9457 problem detail

### Requirement: Ad validation
The API SHALL reject a create or edit request whose `title` is blank or longer than 100 characters, or whose `description` is blank or longer than 2000 characters.

#### Scenario: Blank title
- **WHEN** a client posts or puts an ad with a blank `title`
- **THEN** the response is `400 Bad Request` with an RFC 9457 problem detail and no ad is created or changed

### Requirement: Cross-origin access
The API SHALL allow `GET`, `POST` and `PUT` requests to `/api/**` from configured frontend origins, by default `http://localhost:5173`.

#### Scenario: Frontend preflight
- **WHEN** the browser sends a CORS preflight from `http://localhost:5173`
- **THEN** the response allows that origin

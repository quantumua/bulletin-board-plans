# user-auth Specification

## Purpose

Lets people create a bulletin board account with an email address and password, sign in and out, and lets clients find out who is currently signed in.

## Requirements

### Requirement: Register with email and password
The system SHALL allow a visitor to create an account by providing an email address and a password. On success the system SHALL create the account, sign the new user in, and return the account's id and email.

#### Scenario: Successful registration
- **WHEN** a visitor registers with email `alice@example.com` and a valid password
- **THEN** the system responds with status 201 and the new account's id and email
- **AND** subsequent requests from that client are treated as signed in as `alice@example.com`

### Requirement: Registration input validation
The system SHALL reject registration when the email is missing, blank, not a syntactically valid address, or longer than 254 characters, or when the password is shorter than 8 characters or longer than 72 bytes when UTF-8 encoded. Rejections SHALL respond with status 400 and a problem-details body, and SHALL NOT create an account.

#### Scenario: Invalid email
- **WHEN** a visitor registers with email `not-an-email`
- **THEN** the system responds with status 400 and a problem-details body
- **AND** no account is created

#### Scenario: Password too short
- **WHEN** a visitor registers with a 7-character password
- **THEN** the system responds with status 400 and a problem-details body
- **AND** no account is created

#### Scenario: Non-ASCII password over the byte limit
- **WHEN** a visitor registers with a 42-character Cyrillic password (84 bytes in UTF-8)
- **THEN** the system responds with status 400 and a problem-details body
- **AND** no account is created

### Requirement: Email addresses are unique and case-insensitive
The system SHALL treat email addresses as case-insensitive and ignore surrounding whitespace, storing them normalized to lower case. Registering an email that already belongs to an account SHALL respond with status 409 and a problem-details body.

#### Scenario: Duplicate email with different case
- **WHEN** an account exists for `alice@example.com`
- **AND** a visitor registers with email ` Alice@Example.COM `
- **THEN** the system responds with status 409 and a problem-details body
- **AND** no second account is created

### Requirement: Passwords are never exposed
The system SHALL store passwords only as salted one-way hashes and SHALL NOT include a password or password hash in any response.

#### Scenario: Responses omit password data
- **WHEN** a user registers, signs in, or requests the current user
- **THEN** the response body contains no password or password-hash field

### Requirement: Sign in with email and password
The system SHALL sign a user in when the submitted email (case-insensitive) and password match an account, and return the account's id and email.

#### Scenario: Successful sign-in
- **WHEN** a user with account `alice@example.com` signs in with `ALICE@example.com` and the correct password
- **THEN** the system responds with status 200 and the account's id and email
- **AND** subsequent requests from that client are treated as signed in

### Requirement: Failed sign-in does not reveal which credential was wrong
The system SHALL respond to a sign-in with an unknown email or a wrong password with status 401 and the same problem-details message, and SHALL NOT sign the client in.

#### Scenario: Wrong password
- **WHEN** a user signs in with a registered email and an incorrect password
- **THEN** the system responds with status 401 and the generic message "Invalid email or password"

#### Scenario: Unknown email
- **WHEN** a visitor signs in with an email that has no account
- **THEN** the system responds with status 401 and the generic message "Invalid email or password"

### Requirement: Sign out
The system SHALL allow a signed-in user to sign out, after which the client SHALL no longer be treated as signed in. Signing out when not signed in SHALL succeed without error.

#### Scenario: Sign out ends the session
- **WHEN** a signed-in user signs out
- **THEN** the system responds with status 204
- **AND** a subsequent current-user request from that client responds with status 401

### Requirement: Current user discovery
The system SHALL let a client ask who is signed in: it SHALL return the account's id and email when the client is signed in, and respond with status 401 when it is not.

#### Scenario: Signed-in client
- **WHEN** a signed-in client requests the current user
- **THEN** the system responds with status 200 and that account's id and email

#### Scenario: Anonymous client
- **WHEN** a client that is not signed in requests the current user
- **THEN** the system responds with status 401

### Requirement: Sign-in persists across page reloads
The web application SHALL keep a user signed in across page reloads until they sign out or the session expires, and SHALL show the signed-in user's email with a Sign out action.

#### Scenario: Reload while signed in
- **WHEN** a signed-in user reloads the web application
- **THEN** the application shows them as signed in with their email, without asking for credentials again

### Requirement: Web application sign-in and registration screens
The web application SHALL provide a Sign in form and a Register form, each taking an email and password, and SHALL display the server's error message when the submission fails.

#### Scenario: Register from the web application
- **WHEN** a visitor fills in the Register form with a new email and valid password and submits it
- **THEN** the application shows them as signed in with that email

#### Scenario: Failed sign-in shows error
- **WHEN** a user submits the Sign in form with a wrong password
- **THEN** the application shows "Invalid email or password" and remains signed out

### Requirement: Ads remain available anonymously
Signing in SHALL NOT be required to list, view, create, or edit ads; ad behavior SHALL be identical for signed-in and anonymous clients.

#### Scenario: Anonymous ad creation still works
- **WHEN** a client that is not signed in creates an ad with a valid title and description
- **THEN** the system creates the ad and responds with status 201

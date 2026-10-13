## Why

Logged-in users have no way to see who else is on the bulletin board. A separate "Users" tab listing registered accounts makes the community visible and gives a natural place to see when people joined.

## What Changes

- New endpoint `GET /api/users` returning all registered users (`id`, `username`, `createdAt`), newest registration first, for logged-in users only.
- Accounts record their registration time (`createdAt`); passwords and hashes are never exposed.
- The frontend gets a separate "Users" tab next to the ads view, shown to logged-in users only.
- Existing endpoints and the register/login responses are unchanged.

## Capabilities

### New Capabilities
- `user-directory`: listing registered users (representation, access control, ordering) for display in the UI.

### Modified Capabilities
<!-- None: user-auth requirements are unchanged; the new endpoint lives in its own capability. -->

## Impact

- Backend: new `GET /api/users` endpoint, `createdAt` column on `users`, security rule requiring login for the path.
- Frontend: tab navigation, users list view, hidden for anonymous visitors.
- Privacy: exposes usernames and registration time of all accounts to any logged-in user (accepted by the requester).

## Repos

- **bulletin-board-backend** → local change `add-users-list-backend`: `createdAt` on accounts, `GET /api/users`, login-required rule
- **bulletin-board-frontend** → local change `add-users-list-frontend`: "Users" tab, list view, login-gated visibility

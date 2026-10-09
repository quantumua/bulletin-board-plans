# Tasks

This list holds only contract and coordination work. Backend and frontend each plan their implementation tasks in their own local change, which cites the `user-auth` and `ads-api` specs from this store.

## 1. Contract (this store)

- [x] 1.1 Run `openspec validate add-login --strict --store team-plans` and verify it passes
- [x] 1.2 Create a local change in `bulletin-board-backend` and one in `bulletin-board-frontend` that reference this change, and verify each shows up in `openspec list`
- [x] 1.3 Fix the README store id (`team-plans` vs `bulletin-board-plans`) and verify the documented commands run as written

## 2. Per-repo changes

Each repo plans and tracks its own implementation in a local change that cites this change's specs. Tick these when the repo has archived its change.

- [x] 2.1 `bulletin-board-backend`: local change `add-login-backend` archived
- [x] 2.2 `bulletin-board-frontend`: local change `add-login-frontend` archived

## 3. Integration

- [x] 3.1 With both apps running, register, log in, create an ad, edit it, log out, and verify anonymous read still works and edit is no longer possible
- [x] 3.2 Verify a second user cannot edit the first user's ad and that a request without the CSRF header gets 403 from the browser flow

# Tasks

This list holds only contract and coordination work. Backend and frontend each plan their implementation tasks in their own local change, which cites the `user-directory` spec from this store.

## 1. Contract (this store)

- [x] 1.1 Run `openspec validate add-users-list --strict --store team-plans` and verify it passes
- [x] 1.2 Create a local change in `bulletin-board-backend` and one in `bulletin-board-frontend` that reference this change, and verify each shows up in `openspec list`

## 2. Per-repo changes

Each repo plans and tracks its own implementation in a local change that cites this change's specs. Tick these when the repo has archived its change.

- [x] 2.1 `bulletin-board-backend`: local change `add-users-list-backend` archived
- [x] 2.2 `bulletin-board-frontend`: local change `add-users-list-frontend` archived

## 3. Integration

- [x] 3.1 With both apps running, register two users, log in, open the Users tab, and verify both appear newest first with registration dates
- [x] 3.2 Verify the Users tab is hidden when anonymous and `GET /api/users` without a session returns 401

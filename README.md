# bulletin-board-plans

[OpenSpec](https://github.com/Fission-AI/OpenSpec) store holding the shared planning for the bulletin board application.

| Repo | Role |
|------|------|
| [bulletin-board-backend](https://github.com/quantumua/bulletin-board-backend) | Implements the API specified here |
| [bulletin-board-frontend](https://github.com/quantumua/bulletin-board-frontend) | Consumes the API specified here |

Shared contracts live in `openspec/specs/` (start with `ads-api`). Changes to a contract are planned in
`openspec/changes/` here; each code repo then plans its own implementation change locally and references this store.

## Setup (once per machine)

Requires OpenSpec 1.14+ (`npm install -g @fission-ai/openspec@latest`).

```bash
git clone https://github.com/quantumua/bulletin-board-plans.git ~/Code/bulletin-board-plans
openspec store register ~/Code/bulletin-board-plans --id team-plans
```

## Typical flow

1. Plan the contract change here. Describe each affected repo in the proposal's "Repos" section.

   ```bash
   openspec new change <change-id> --store team-plans
   openspec validate <change-id> --strict --store team-plans
   ```

2. In each code repo, create the local change named in "Repos" (`<change-id>-<repo>`). The repo's
   `openspec/config.yaml` tells `/opsx:propose` to read this change, cite its specs and plan only that repo's work.
   Read the change and its spec deltas with:

   ```bash
   openspec show <change-id> --store team-plans
   openspec show ads-api --type spec --store team-plans
   ```

3. Implement and archive the change in each repo. Record any contract gap a repo had to decide in this change's
   `design.md` "Clarifications" section, by pull request.
4. Tick that repo's checkpoint in this change's "Per-repo changes" tasks.
5. When every checkpoint is ticked, archive the store change last: `openspec archive <change-id> --store team-plans`.

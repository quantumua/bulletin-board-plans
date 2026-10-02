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
openspec store register ~/Code/bulletin-board-plans --id bulletin-board-plans
```

## Typical flow

```bash
openspec new change <change-id> --store bulletin-board-plans   # plan the shared contract here
openspec validate --store bulletin-board-plans
```

Then, in each code repo, create a local change that implements its part and cites the store spec
(`openspec show ads-api --type spec --store bulletin-board-plans`).

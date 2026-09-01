# Contributing

Payflow is three repositories. Each has its own `CONTRIBUTING.md` with the full
detail; this page covers what applies everywhere.

| Repo | Stack | CI gates |
|---|---|---|
| `payflow-contract` | Rust, soroban-sdk 26 | fmt, clippy, test, wasm build |
| `payflow-backend` | TypeScript, Node 22 | typecheck, test, build |
| `payflow-frontend` | Next.js 15, React 19 | typecheck, build |

## Picking work

Issues are labelled by complexity, matching the Drips Wave point tiers:

| Label | Points |
|---|---|
| `complexity:trivial` | 100 |
| `complexity:medium` | 150 |
| `complexity:high` | 200 |

Start with `good-first-issue`. Comment on an issue before starting so two people
do not build the same thing.

## Rules that apply to every repo

- **Conventional commits**, one logical change per commit:
  `feat(vault): support batch deposits`
- **Never `git add .`** — stage the specific files you changed.
- **Money is never a float.** `i128` in Rust, `bigint` in TypeScript, `TEXT` in
  SQLite. Proportions are integer basis points.
- **Tests are not optional.** Cover the error path, not just the happy path.
- **Run CI locally before pushing.** The exact commands are in each repo's
  contributing guide.

## Cross-repo changes

Some work spans repos — a new contract function usually needs indexer support
and UI. Open an issue in each, and cross-reference with an explicit
`Depends on: <org>/<repo>#<n>`.

Order matters. Ship in dependency order: contract, then indexer, then UI.
Merging a frontend feature before its contract is deployed produces bugs that
look like regressions.

## Security issues

Do not open a public issue. Use GitHub private vulnerability reporting on the
affected repository. See `SECURITY.md` in each repo for scope.

# Design — initial-version

## New files

| File | Purpose |
|------|---------|
| `README.md` | Entry point: overview, repo map table, how-it-works summary |
| `architecture.md` | Dependency graph, execution loop, data flow, extension points |
| `concepts.md` | PSMs, knowledge files, incremental formalization, auditability, provider independence |
| `getting-started.md` | CLI quickstart, Docker API guide, new domain walkthrough |
| `CONTRIBUTING.md` | Scope of this repo, style guide, what belongs elsewhere |
| `package.json` | npm manifest with `@commonkads/dev-workflow` as a dev dependency |
| `.npmrc` | Points `@commonkads` scope to GitHub Package Registry |

The `LICENSE`, `CLAUDE.md`, `docs/` (workflow docs), `.claude/` (feature skill), `node_modules/`, `package-lock.json` are all produced by the GitHub repo creation and `npm install` — not authored content.

## Key decisions

**Repo map format** — use a Markdown table (not a prose list) in `README.md` so it stays scannable as the number of repos grows.

**architecture.md dependency graph** — ASCII art tree rather than a Mermaid diagram, so it renders correctly everywhere without a diagram renderer. `api-author` appears under `author` (not under `engine` directly) since it depends on `author`, which in turn depends on `engine`.

**concepts.md knowledge files** — the original draft described norms, abstraction, evaluation, and selection as universal "layers." These are Assessment-PSM-specific file types, not a universal schema. The design uses a table of Assessment example files, clearly scoped as an example of one PSM's files, not a mandatory structure.

**concepts.md auditability** — the original draft claimed exact reproducibility of results. This is only true for deterministic steps. LLM steps have no reproducibility guarantee; the audit log provides explanation, not replay. The final text distinguishes these cases explicitly.

**concepts.md PSM definition** — a PSM is defined as "the implementation steps of a knowledge task," not merely "a reasoning template," reflecting the CommonKADS methodology framing.

**package.json** — `private: true` since this repo is never published to a package registry. Only `@commonkads/dev-workflow` is needed as a dev dependency; no runtime dependencies.

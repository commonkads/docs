# Requirements — initial-version

## In scope

- A `README.md` serving as the entry point: what CommonKADS is, a repo map, and quick navigation to the other docs.
- An `architecture.md` covering the full dependency graph, execution loop, data flow, and extension points — the cross-repo view that no individual README provides.
- A `concepts.md` explaining PSMs (as the implementation steps of a knowledge task), knowledge files (universal vs. PSM-specific), incremental formalization, auditability, and provider independence.
- A `getting-started.md` with step-by-step guides for: running an example domain via the CLI, deploying `api-runtime` with Docker, and creating a new knowledge domain from scratch.
- A `CONTRIBUTING.md` defining what belongs in this repo vs. individual repos, and the review process for doc changes.
- A `package.json` and `.npmrc` that install `@commonkads/dev-workflow` as a dev dependency, enabling the feature workflow and shared docs in this repo.
- Initialize the repo as a git repository, merge the GitHub-created `LICENSE`, and leave it ready to push.

## Out of scope

- Per-repo API references (live in each package's own `README.md` or `docs/`).
- Requirements and design specs for individual packages (live in each repo's `docs/specs/`).
- A GitHub Pages site or any static site generator setup.
- Any automation (CI, publish workflow).
- Content for `api-author` beyond a placeholder (the package is a stub).

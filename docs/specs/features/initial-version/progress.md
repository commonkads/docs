# Progress — initial-version

## 2026-11-27

- Decision: used a dedicated `docs` repo rather than adding docs to an existing repo or using a GitHub wiki — keeps documentation lifecycle independent of any single codebase and allows cross-repo guides to live naturally in one place.
- Decision: `private: true` in `package.json` — this repo is never published to a package registry, only deployed as a GitHub repo.
- Deviation: `api-author` was initially drawn as a direct branch off `engine` in the architecture dependency graph. Corrected to show it only under `author` after user pointed out that `api-author` depends on `author`, not directly on `engine`. See `07-changes.md`.
- Deviation: `concepts.md` initially described norms, abstraction, evaluation, and selection as universal "knowledge layers." User corrected this — those are Assessment-PSM-specific file types, not a universal schema. Section rewritten to separate universal files from PSM-specific ones. See `07-changes.md`.
- Deviation: `concepts.md` auditability section initially claimed exact reproducibility of results for all steps. User corrected this — LLM steps have no reproducibility guarantee; the audit log provides explanation, not replay. Text updated to distinguish the two cases explicitly. See `07-changes.md`.
- Decision: merged GitHub's initial commit (LICENSE only) using `--allow-unrelated-histories` since the local repo and GitHub repo were initialized independently with separate root commits.
- Decision: approved `@commonkads/dev-workflow` postinstall script via `npm install-scripts approve` — this is the standard pattern used across all other repos in the project.

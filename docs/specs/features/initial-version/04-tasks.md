# Tasks — initial-version

Tasks are listed in execution order. All are complete.

1. **Survey all 12 repos** — read each `README.md` and `CLAUDE.md` to understand purpose, dependencies, and CLI/API surfaces. Output: mental model needed for all subsequent tasks.

2. **Write `README.md`** — entry point with one-paragraph overview, repo map table (core / APIs / providers / knowledge+tooling), and a "how it works at a glance" numbered summary.

3. **Write `architecture.md`** — ASCII dependency graph, layer descriptions (knowledge / engine / provider / deployment), execution loop diagram, data flow walkthrough, extension points table. Ensure `api-author` is correctly placed under `author` in the graph.

4. **Write `concepts.md`** — why CommonKADS, PSMs (definition + three built-in templates with step tables), knowledge files (universal vs. PSM-specific), incremental formalization, auditability (distinguishing deterministic vs. LLM steps), provider independence.

5. **Write `getting-started.md`** — three guides: (a) CLI with `runner` + `knowledge-examples`, (b) Docker deploy of `api-runtime` with environment variable table, (c) creating a new domain from scratch including `domain.yml` structure and `author` validation.

6. **Write `CONTRIBUTING.md`** — what belongs in this repo vs. individual repos, style rules (American English, present tense, runnable code examples, ASCII diagrams).

7. **Initialize git repo** — `git init`, rename branch to `main`, add remote (`git@github-commonkads:commonkads/docs.git`), fetch origin, merge GitHub's initial commit (LICENSE only) with `--allow-unrelated-histories`.

8. **Add `package.json` and `.npmrc`** — `private: true` manifest with `@commonkads/dev-workflow: ^1.3.0` dev dependency; `.npmrc` pointing `@commonkads` scope to `https://npm.pkg.github.com`.

9. **Run `npm install`** — approve and run postinstall script; confirms it copies workflow docs into `docs/` and creates `CLAUDE.md`.

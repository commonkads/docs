# Brainstorm — initial-version

## Problem

The CommonKADS project spans 12 repositories. No single place documents how they fit together, how to get started, or what the core concepts mean. Each repo's README covers only that repo.

## Options explored

**Option A — docs in an existing repo**  
Add a top-level `docs/` to `engine` or `runner`. Keeps things simple but ties documentation lifecycle to a specific codebase. Breaks when the docs cover things that cross repos.

**Option B — GitHub wiki on a hub repo**  
GitHub wikis are not version-controlled alongside code, cannot be linted or reviewed via PRs, and are harder to evolve collaboratively. Ruled out.

**Option C — dedicated `docs` repo**  
A standalone repo owned by the `commonkads` org. Docs have their own issue tracker, PR review, and release cadence. Cross-repo guides live here naturally. Can optionally publish as a GitHub Pages site later.

## Direction chosen

Option C. The `docs` repo serves as the cross-repo home: architecture overview, concepts, and getting-started guides. Per-repo API references, coding standards, and design specs stay in each individual repo.

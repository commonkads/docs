# Contributing

## Repository overview

This `docs` repo is the cross-repo documentation home for the CommonKADS project. For code contributions, see the individual repos listed in the [README](README.md).

## Editing these docs

1. Fork this repo and create a branch.
2. Edit the relevant Markdown file(s) in the root.
3. Open a pull request with a short description of what changed and why.

## What belongs here

- Architecture and repo relationship diagrams
- Getting-started guides and tutorials
- Conceptual explanations of PSMs, knowledge layers, and design decisions
- Cross-repo migration or upgrade guides

## What belongs in individual repos

- API references and interface documentation → each package's own `README.md` or `docs/`
- Coding standards, CI guidance, and developer workflow → installed from `@commonkads/dev-workflow`
- Requirements and design specs → each repo's `docs/specs/`

## Style

- American English
- Present tense, active voice
- Code examples should be runnable — test them before including them
- Diagrams as ASCII (inline in Markdown); no binary image files unless necessary

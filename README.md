# CommonKADS (Platform)

> **Disclaimer:** This project is an independent open-source software implementation inspired by the CommonKADS knowledge engineering methodology (Schreiber et al.). It is not affiliated with or endorsed by the original methodology authors or any third-party trademark holders.

CommonKADS is a TypeScript platform for building knowledge-intensive applications. It separates domain knowledge from code and uses an LLM at runtime to reason over that knowledge and produce auditable, traceable results. Domain models (rules, decision tables, etc.) and ontologies (data types and relationships) live as plain text files authored by domain experts. It is grounded in the CommonKADS knowledge engineering methodlogy.

## Quick navigation

| Goal | Start here |
|------|-----------|
| Understand the architecture | [architecture.md](architecture.md) |
| Run your first domain | [getting-started.md](getting-started.md) |
| Learn the core concepts | [concepts.md](concepts.md) |
| Create a new knowledge domain | [getting-started.md#creating-a-new-domain](getting-started.md#creating-a-new-domain) |
| Deploy the REST API | [getting-started.md#deploying-the-rest-api](getting-started.md#deploying-the-rest-api) |

## Repository map

### Core

| Repo | npm package | Purpose |
|------|-------------|---------|
| [engine](https://github.com/commonkads/engine) | `@commonkads/engine` | Task execution loop, PSM templates, provider interfaces |
| [runner](https://github.com/commonkads/runner) | `@commonkads/runner` | CLI for running a domain against a live LLM |
| [author](https://github.com/commonkads/author) | `@commonkads/author` | Authoring tools: validate knowledge files and run test cases |

### APIs

| Repo | Purpose |
|------|---------|
| [api-runtime](https://github.com/commonkads/api-runtime) | Docker REST API — HTTP wrapper around the engine (planned) |
| [api-author](https://github.com/commonkads/api-author) | Docker REST API for knowledge authoring (planned) |
| [app-studio](https://github.com/commonkads/app-studio) | Single-page app for knowledge management and testing; calls `api-author` and `api-runtime` (planned) |

### Providers

| Repo | npm package | LLM |
|------|-------------|-----|
| [provider-anthropic](https://github.com/commonkads/provider-anthropic) | `@commonkads/provider-anthropic` | Anthropic Claude |
| [provider-aws](https://github.com/commonkads/provider-aws) | `@commonkads/provider-aws` | AWS Bedrock + S3 |
| [provider-google](https://github.com/commonkads/provider-google) | `@commonkads/provider-google` | Google Gemini |
| [provider-openai](https://github.com/commonkads/provider-openai) | `@commonkads/provider-openai` | OpenAI |
| [provider-ollama](https://github.com/commonkads/provider-ollama) | `@commonkads/provider-ollama` | Ollama (local) |

### Knowledge and tooling

| Repo | npm package | Purpose |
|------|-------------|---------|
| [knowledge-examples](https://github.com/commonkads/knowledge-examples) | — | Ready-to-run example domains |
| [dev-workflow](https://github.com/commonkads/dev-workflow) | `@commonkads/dev-workflow` | Shared developer workflow docs, installed into each repo |

## How it works at a glance

1. A **knowledge domain** is a directory of text files: a `domain.yml` manifest, an ontology, norms, abstraction rules, and evaluation criteria — all written by domain experts, no code required.
2. The **engine** loads those files at runtime, selects the right PSM template (Assessment, Diagnosis, or Monitoring), and drives a fixed reasoning sequence.
3. At each inference step the engine sends the relevant knowledge text plus the current case data to an **LLM provider**, which returns a structured JSON result.
4. Every step logs the exact document versions and reasoning used, giving a complete **audit trail**.
5. The **runner** CLI or **api-runtime** REST API are the two deployment surfaces.

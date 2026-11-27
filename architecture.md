# Architecture

## Repository dependency graph

```
knowledge-examples  ← (no code dependency — knowledge files only)
       │
       ▼
   engine  ◄──────────────────────────────── provider-*
       │                                    (register at startup)
       ├──► runner (CLI)
       │
       ├──► author (CLI, planned)
       │       │
       │       └──► api-author (Docker REST API, planned)
       │                   │
       │                   ├──────────────────────────────► app-studio (SPA, planned)
       │                                                        ▲
       └──► api-runtime (Docker REST API, planned)  ─────────────┘

dev-workflow  ← npm install copies docs into each repo above
```

**`engine`** is the only required dependency. Everything else is optional depending on your deployment model.

## Layers

### Knowledge layer (no code)

A domain lives entirely in text files. The engine reads them at runtime — no compilation, no code changes needed to update domain logic.

```
<domain-root>/
├── domain.yml              ← manifest: name, PSM type, input/output schema, knowledge paths
├── ontology/
│   └── domain.yml          ← entity types, relationships, enumerations
└── <topic>/
    └── <psm>/
        ├── norms.yml       ← thresholds, rules, norms
        ├── abstraction.md  ← how to map raw observations to abstract concepts
        ├── evaluation.md   ← how to judge the abstractions
        └── selection.md    ← optional: filters norms to those relevant to this case
```

### Engine layer

**`RuntimeEngine`** (`@commonkads/engine`) is the orchestrator. Its execution loop:

```
execute(psm, input, context)
  │
  ├─ psm.initializeState(input)
  │
  └─ loop until psm.getNextStep() returns null:
       ├─ fetch knowledge files referenced by this step (parallel)
       ├─ if deterministic step → pass knowledge text back to PSM directly
       └─ if LLM step → build prompt → call InferenceProvider → fold result into state
  │
  └─ psm.mapOutput(state) → final result
```

**PSM templates** encode fixed reasoning sequences. The three built-in templates are:

| Template | Steps | Typical use |
|----------|-------|------------|
| Assessment | obtain → specify → abstract → evaluate → conclude | "Is this observation valid/within range?" |
| Diagnosis | observe → hypothesise → verify → conclude | "What is the root cause of this problem?" |
| Monitoring | observe → compare → evaluate → conclude | "Is this metric within threshold?" |

Domain subclasses override only the specific step hooks they need. Deterministic steps (like filtering norms to those relevant to the current case) run as plain code; inference steps are handed to the LLM.

### Provider layer

Providers implement three interfaces defined in `@commonkads/engine`:

| Interface | Purpose | Built-in |
|-----------|---------|---------|
| `IInferenceProvider` | Send a prompt, receive structured JSON | Ollama (via `@commonkads/provider-ollama`) |
| `IFileStore` | Resolve knowledge files by path | `LocalFileStore` (built into engine) |
| `IKnowledgeStore` | Semantic search over knowledge passages | stub (future) |

Cloud and commercial LLM providers are in separate packages. Register them at startup:

```ts
import { registerAnthropicProviders } from '@commonkads/provider-anthropic';
registerAnthropicProviders();
// then buildInference({ type: 'claude', model: '...' }) works
```

The same pattern applies to `@commonkads/provider-aws`, `@commonkads/provider-google`, and `@commonkads/provider-openai`.

### Deployment layer

Two deployment surfaces:

**CLI — `runner`**  
Direct process execution. Best for scripts, CI pipelines, or local experimentation.

```
runner --knowledge-root ./knowledge/vital-signs \
       --payload '{"PatientType":"adult","HeartRate":72,...}' \
       --provider ollama --model llama3
```

**REST API — `api-runtime`**  
Docker container. Best for production services that need an HTTP interface.

```
POST /execute
{ "domain": "vital-signs", "input": { "PatientType": "adult", ... } }
```

The Docker image is built with a `PROVIDERS` build arg that selects which provider packages to include, keeping image size small.

## Data flow — end to end

```
Client (HTTP or CLI)
        │
        │  domain + input JSON
        ▼
  runner / api-runtime
        │
        │  loads domain.yml → builds catalog
        │  builds InferenceProvider
        ▼
   RuntimeEngine.execute()
        │
        ├─ step: obtain (deterministic)
        │   └─ reads ontology + input → initial state
        │
        ├─ step: specify (deterministic, may filter norms)
        │   └─ narrows knowledge scope to this case
        │
        ├─ step: abstract (LLM)
        │   ├─ reads abstraction.md + norms
        │   ├─ calls InferenceProvider.infer(prompt)
        │   └─ parses JSON → state.abstractions
        │
        ├─ step: evaluate (LLM)
        │   ├─ reads evaluation.md
        │   ├─ calls InferenceProvider.infer(prompt)
        │   └─ parses JSON → state.evaluation
        │
        └─ step: conclude (deterministic)
            └─ maps state → final output JSON
        │
        ▼
  result + optional audit log / explain / timings
```

## Audit and observability

Every non-deterministic step records:

- The knowledge document ids and versions used to build the prompt
- The LLM's raw response (when `--explain` is set)
- Step wall-clock timing

This means any result can be fully reproduced and explained after the fact, even as knowledge files evolve.

## Extension points

| What to extend | How |
|----------------|-----|
| New LLM provider | Implement `IInferenceProvider`; call `registerProvider()` at startup |
| New cloud file store | Implement `IFileStore` or `IStorageBackend` |
| New PSM reasoning sequence | Extend `BasePsm`; declare a new `stepNames` list |
| New knowledge domain | Write domain files + a PSM subclass; no engine changes needed |

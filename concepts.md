# Concepts

## Why CommonKADS?

Most LLM applications embed domain logic in code or in prompts baked into the application. This creates several problems:

- Domain experts cannot update rules without developer involvement.
- It is hard to audit why a decision was made.
- Rules and code evolve on the same cadence, increasing deployment coupling.

CommonKADS solves this by separating three concerns that should evolve independently:

| Concern | Owner | Cadence |
|---------|-------|---------|
| Domain knowledge (rules, ontology) | Domain experts | Continuous |
| Reasoning structure (PSM templates) | Knowledge engineers | Infrequent |
| Execution infrastructure (engine, providers) | Developers | Infrequent |

## Problem-Solving Methods (PSMs)

A PSM defines the implementation steps of a knowledge task — the control structure that drives how a particular class of problem is solved. That structure can include loops, conditionals, and decision points, but the PSM logic itself is kept deliberately simple: branching decisions and domain reasoning live in the inference steps and the domain knowledge that feeds them, not in the PSM. The same PSM serves every domain in its class; only the knowledge files change.

This CommonKADS platform (currently) provides three built-in templates:

### Assessment
> "Is this observation valid, within range, or meets criteria?"

Steps: **obtain → specify → abstract → evaluate → conclude**

- **obtain** — load the ontology and normalize the input
- **specify** — filter the norms to those relevant to this case (e.g., adult vs. pediatric norms)
- **abstract** — LLM maps raw observations to abstract categories (normal, elevated, critical)
- **evaluate** — LLM applies the evaluation criteria to the abstractions and produces a verdict
- **conclude** — deterministically assembles the final output

Example domains: vital signs validity, lab value assessment, compliance checking.

### Diagnosis
> "What is the root cause of this problem?"

Steps: **observe → hypothesise → verify → conclude**

- **observe** — load the fault catalog and normalize the symptoms
- **hypothesise** — deterministically generate candidate hypotheses from the catalog
- **verify** — LLM evaluates which hypotheses are supported by the observations
- **conclude** — select the most likely root cause

Example domains: automotive fault diagnosis, infrastructure incident triage.

### Monitoring
> "Is this metric within threshold, and what is its status?"

Steps: **observe → compare → evaluate → conclude**

- **observe** — load the threshold definitions
- **compare** — deterministically compare observations against thresholds
- **evaluate** — LLM interprets the comparison results in context (trends, severity)
- **conclude** — produce a status and any alerts

Example domains: server monitoring, IoT sensor thresholds, SLA tracking.

### Why PSM templates improve LLM quality

Decomposing a task into focused steps has a direct effect on reasoning quality. Each step asks the LLM a narrow, well-scoped question — "abstract these observations into categories" or "evaluate these abstractions against these criteria" — rather than asking it to solve the entire problem in one pass. A focused prompt with a small, relevant slice of knowledge is easier to reason about correctly than a broad prompt that mixes multiple concerns.

This has two practical benefits: first, simpler and smaller models can be used for individual steps without sacrificing accuracy, since each step's complexity is bounded; second, outcomes tend to be more consistent across runs, because there is less ambiguity about what the model is being asked to do and less opportunity for the model to conflate different aspects of the problem.

## Knowledge models

A knowledge model is the collection of knowledge that describes a domain. The engine retrieves knowledge at runtime from a catalog, which abstracts over the underlying storage — a local directory, object storage (S3, GCS), a relational database, or any other backend that implements the catalog interface. The structure described here uses the local file store as the concrete example; other backends organize the same content differently.

### Model structure (local file store)

Every model starts with a **`model.yml`** file — the manifest that declares the model's identity, its PSM template, its input/output schema, and references to the knowledge it contains.

A model can then have any combination of three kinds of knowledge, each in its own subdirectory:

- **`ontology/`** — zero or more ontology files  
  An ontology defines the vocabulary of a domain: the entity types, their attributes, enumerations, and the relationships between them. It is the shared conceptual model that all other knowledge in the domain refers to. Ontology files can be authored in multiple formats depending on the required level of formalization:
  - **Markdown** — prose descriptions of concepts and relationships; readable by domain experts and LLMs alike
  - **YAML** — structured entity and relationship definitions; suitable for most practical domains
  - **Turtle (`.ttl`)** — W3C standard serialization of [RDF](https://www.w3.org/RDF/) triples; enables formal reasoning and interoperability with semantic web tooling ([OWL](https://www.w3.org/OWL/), [Turtle spec](https://www.w3.org/TR/turtle/))
  - **SBVR Fact Model** — a business-oriented vocabulary and fact type model based on the [OMG SBVR standard](https://www.omg.org/spec/SBVR/); focuses on natural-language-grounded concept definitions

- **`task/`** — zero or more task definitions  
  A task ties a PSM template to a set of knowledge references. It declares what input the task accepts, what output it produces, and which inference knowledge and ontology entries each step should load. A model may define multiple tasks (e.g., an assessment task and a monitoring task over the same ontology).

- **`inference/`** — zero or more inference knowledge files  
  Inference knowledge captures how to reason about data: rules, decision tables, norms, abstraction instructions, evaluation criteria, and similar guidance. These files are what the engine hands to the LLM (or evaluates deterministically) at each inference step. Supported formats include:
  - **Markdown** — natural language rules and guidance; authored by domain experts, consumed directly by the LLM
  - **YAML** — structured rules and decision tables; can be processed deterministically before or instead of an LLM call
  - **SBVR Business Rules** — formally stated rules grounded in the SBVR vocabulary ([OMG SBVR standard](https://www.omg.org/spec/SBVR/)); rules are expressed as normative statements ("It is obligatory that…", "It is prohibited that…") tied to the concepts defined in the SBVR Fact Model, providing machine-readable precision while remaining readable by business stakeholders

```
<model-root>/
├── model.yml
├── ontology/
│   └── *.yml / *.md / *.ttl
├── task/
│   └── *.yml
└── inference/
    └── *.yml / *.md
```

The engine does not prescribe file names or how many files of each kind a model has — `model.yml` declares the references, and the catalog resolves them from whatever storage backend is in use.

## Incremental formalization

Knowledge does not need to be fully formalized upfront. CommonKADS supports a gradual path:

1. **Plain text** — write knowledge files as natural language descriptions. The LLM can reason from them immediately.
2. **Structured YAML** — migrate norms and ontology into YAML as the domain stabilizes. Deterministic steps can then filter and validate them.
3. **Formal ontology** — migrate toward SBVR (Semantics of Business Vocabulary and Rules) or OWL/RDF as compliance and interoperability requirements grow.

Each step improves traceability and allows more deterministic processing, without requiring a big-bang migration.

LLMs also tend to produce more consistent and accurate results as knowledge becomes more formal. Plain text leaves room for ambiguity — an LLM must interpret vague terms and may do so differently across calls. Structured YAML and formal notations like SBVR and OWL eliminate that ambiguity by making concepts, relationships, and constraints explicit and unambiguous. The more precisely the domain is specified, the less the LLM has to guess, and the more reliably it reasons.

## Auditability

Every non-deterministic (LLM) step records:

- The id and version of each knowledge document that was included in the prompt
- The LLM's response (when `--explain` is set)
- Wall-clock timing for the step

For deterministic steps, re-running with the same document versions will produce the same result. For LLM steps there is no such guarantee — the same prompt can yield a different response. What the audit log provides is explanation: the exact knowledge documents and versions that were used, and (when `--explain` is set) the LLM's reasoning at each step, so a result can always be understood even if it cannot be exactly reproduced.

## Provider independence

The engine defines three small interfaces (`IInferenceProvider`, `IFileStore`, `IKnowledgeStore`). Provider packages implement these interfaces and register themselves at startup. Swapping from Ollama to Claude to Bedrock requires only a configuration change — no changes to domain code or knowledge files.

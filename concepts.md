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

A PSM is a reusable reasoning template that encodes a fixed sequence of inference steps. The sequence is the same for every domain that uses a given template — only the knowledge files change.

CommonKADS provides three built-in templates:

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

## Knowledge layers

A domain's knowledge is organized into layers that can be updated independently:

### Ontology layer
Defines the vocabulary: entity types, enumerations, and relationships. Lives in `ontology/domain.yml`. Changes here require no code changes — just update the YAML.

### Norm layer
Defines thresholds, rules, and reference values. Lives in `norms.yml`. Domain experts can add, remove, or change norms without touching any other file.

### Abstraction layer
Instructions (plain English) for how to map raw observations to abstract concepts. The LLM reads this at inference time. Lives in `abstraction.md`.

### Evaluation layer
Instructions for how to interpret the abstractions and reach a verdict. Lives in `evaluation.md`.

### Selection layer (optional)
When a domain has many norms but only some apply to a given case (e.g., pregnancy-specific ranges), a `selection.md` or `selection.yml` filters the norm set deterministically before the LLM step.

## Incremental formalization

Knowledge does not need to be fully formalized upfront. CommonKADS supports a gradual path:

1. **Plain text** — write knowledge files as natural language descriptions. The LLM can reason from them immediately.
2. **Structured YAML** — migrate norms and ontology into YAML as the domain stabilizes. Deterministic steps can then filter and validate them.
3. **Formal ontology** — migrate toward SBVR (Semantics of Business Vocabulary and Rules) or OWL/RDF as compliance and interoperability requirements grow.

Each step improves traceability and allows more deterministic processing, without requiring a big-bang migration.

## Auditability

Every non-deterministic (LLM) step records:

- The id and version of each knowledge document that was included in the prompt
- The LLM's response (when `--explain` is set)
- Wall-clock timing for the step

The audit log means any result can be reproduced exactly — by re-running with the same document versions — and explained — by showing the exact text and reasoning the LLM was given.

## Provider independence

The engine defines three small interfaces (`IInferenceProvider`, `IFileStore`, `IKnowledgeStore`). Provider packages implement these interfaces and register themselves at startup. Swapping from Ollama to Claude to Bedrock requires only a configuration change — no changes to domain code or knowledge files.

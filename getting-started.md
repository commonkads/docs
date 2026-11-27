# Getting started

## Prerequisites

- Node.js 20+
- An LLM provider — Ollama running locally is the easiest starting point; no API key needed.

## Running an example domain

The fastest way to see CommonKADS in action is with the `runner` CLI and the built-in example domains from `knowledge-examples`.

### 1. Clone the repos

```sh
git clone https://github.com/commonkads/runner
git clone https://github.com/commonkads/knowledge-examples
```

### 2. Install and build runner

```sh
cd runner
npm install
npm run build
```

### 3. Run the vital-signs example

```sh
npx runner \
  --knowledge-root ../knowledge-examples/knowledge/vital-signs \
  --payload '{"PatientType":"adult","HeartRate":110,"BloodPressureSystolic":145,"RespiratoryRate":22,"OxygenSaturation":94}' \
  --provider ollama \
  --model llama3
```

Add `--explain` to see the LLM's reasoning at each step, or `--audit` to see which document versions were used.

---

## Deploying the REST API

`api-runtime` is a Docker container that exposes an HTTP interface to the engine.

### 1. Build the image

```sh
git clone https://github.com/commonkads/api-runtime
cd api-runtime

# Local only (Ollama + local file store):
docker build -t api-runtime .

# With Anthropic support:
docker build --build-arg PROVIDERS=anthropic -t api-runtime .

# With AWS Bedrock + S3:
docker build --build-arg PROVIDERS=aws -t api-runtime .
```

Supported `PROVIDERS` values (comma-separated): `aws`, `anthropic`, `google`. Ollama and local file store are always included.

### 2. Run the container

```sh
# Local Ollama example:
docker run -p 3000:3000 \
  -e COMMONKADS_PROVIDER=ollama \
  -e COMMONKADS_MODEL=llama3 \
  -e COMMONKADS_KNOWLEDGE_ROOT=/knowledge \
  -v $(pwd)/../knowledge-examples/knowledge:/knowledge \
  api-runtime
```

```sh
# AWS Bedrock + S3 example:
docker run -p 3000:3000 \
  -e COMMONKADS_PROVIDER=bedrock \
  -e COMMONKADS_MODEL=anthropic.claude-3-5-sonnet-20241022-v2:0 \
  -e COMMONKADS_S3_BUCKET=my-knowledge-bucket \
  -e AWS_REGION=us-east-1 \
  api-runtime
```

### 3. Execute a task

```sh
curl -s http://localhost:3000/execute \
  -H 'Content-Type: application/json' \
  -d '{
    "domain": "vital-signs",
    "input": {
      "PatientType": "adult",
      "HeartRate": 110,
      "BloodPressureSystolic": 145,
      "RespiratoryRate": 22,
      "OxygenSaturation": 94
    }
  }' | jq
```

---

## Creating a new domain

A domain is a directory of text files. No code changes are needed to add a domain — the engine loads it at runtime.

### 1. Create the directory structure

```
my-domain/
├── domain.yml
├── ontology/
│   └── domain.yml
└── <topic>/
    └── <psm>/
        ├── norms.yml
        ├── abstraction.md
        └── evaluation.md
```

### 2. Write domain.yml

`domain.yml` declares the domain name, which PSM template to use, the input/output schema, and where the knowledge files live:

```yaml
name: My Domain
task: assessment
psm: assessment-with-abstraction
input:
  type: object
  required: [field1, field2]
  properties:
    field1:
      type: string
    field2:
      type: number
output:
  type: object
  properties:
    verdict:
      type: string
      enum: [Pass, Review]
knowledge:
  ontologyPath: ontology/domain.yml
  normsPath: topic/psm/norms.yml
  abstractionPath: topic/psm/abstraction.md
  evaluationPath: topic/psm/evaluation.md
```

### 3. Write the knowledge files

**`ontology/domain.yml`** — entity types, enumerations, and relationships in your domain.

**`norms.yml`** — thresholds, ranges, and rules (plain YAML, no code):

```yaml
norms:
  - id: heart-rate-adult
    entity: HeartRate
    context: adult
    low: 60
    high: 100
    unit: bpm
```

**`abstraction.md`** — natural language instructions for the LLM on how to map raw observations to abstract concepts (e.g., "normal", "elevated", "critical").

**`evaluation.md`** — instructions for the LLM on how to evaluate the abstractions and reach a verdict.

### 4. Run your domain

```sh
runner \
  --knowledge-root ./my-domain \
  --payload '{"field1":"value","field2":42}' \
  --provider ollama --model llama3
```

### 5. Validate and test

Use the `author` CLI to check completeness and run test cases:

```sh
author validate --knowledge-root ./my-domain
author test --knowledge-root ./my-domain --provider ollama --model llama3
```

Test cases live in `<domain-root>/tests/*.yml` and specify inputs and expected outputs in YAML — no code needed.

### Going further

- See [knowledge-examples](https://github.com/commonkads/knowledge-examples) for three complete worked domains (vital-signs, car-diagnosis, server-monitoring).
- See [concepts.md](concepts.md) for the reasoning behind the PSM templates and knowledge layers.
- To embed the engine in your own application rather than using the CLI or API, install `@commonkads/engine` and call `RuntimeEngine.execute()` directly.

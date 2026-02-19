# Feature request: triage command surface v2 (mirrors → runs → DocC site)

@Metadata {
  @PageKind(article)
  @TitleHeading("Feature request")
}

Define a clean, decision-grade command surface for `swift-threads-cli` triage generation, based on **disk-canonical mirrors** and **disk-canonical triage runs**, publishing a DocC triage site.

## Context / problems

- The current triage run artifacts are **too thin** (do not always reflect full prioritization/bucketing decisions).
- The site generation is currently reachable via several commands whose intent has drifted.
- We want explicit provenance markers for generated artifacts:
  - generated Markdown: `*.mirror.md`
  - generated JSON: `*.mirror.json`
- We want stable, deterministic naming:
  - `<id>.<type>.mirror.<ext>`
- We want day-level bucket pages to support **multiple triage runs per day**, with a v1 policy:
  - **latest run wins** for day buckets.

## Source of truth (disk)

### Mirrors (canonical)

Thread mirrors live under the operator profile:

- `.clia/profiles/operators/<operator>/threads/{open|incidents|blocked|closed}/<slug>/`

Each thread directory contains at minimum:

- `*.summary.thread.clia.json`

Optionally:

- `*.events.thread.clia.jsonl`

**Buckets are canonical** (`open|incidents|blocked|closed`). Priority is an attribute within a bucket.

### Runs (canonical)

Triage runs are canonical on disk:

- `provisioned/threads/triage/runs/<runId>/`

Run artifacts:

- `triage-run.mirror.json`
- `items.mirror.jsonl`
- `assignment.mirror.json`
- optional `render.mirror.md`

## Desired command surface (v2)

All commands live under the `triage` tree. No compatibility shims.

### `swift-threads-cli triage docc-site …`

This group is responsible for writing the DocC bundle source (pages + resources) from disk.

#### `triage docc-site threads`

Generates stable per-thread pages and resources.

Inputs:
- `--mirrors-root <path>`
- `--routing-map <path>`
- `--docc-root <path>`
- `--tz <iana>`

Outputs (generated):
- `threads/<threadId>.thread.mirror.md`
- `resources/thread-cards/thread-card-<threadId>.svg`

UI rules:
- **Chrome icon** uses badge icons (`thread-icon-pX-sY`).
- **Card image** is the per-thread emoji pair SVG.

#### `triage docc-site views`

Generates hub + daily view pages from mirrors and runs.

Inputs:
- `--mirrors-root <path>`
- `--routing-map <path>`
- `--docc-root <path>`
- `--triage-runs-root <path>`
- `--tz <iana>`

Outputs (generated):
- `threads.mirror.md` (with `@TopicsVisualStyle(detailedGrid)` + `@AutomaticSeeAlso(disabled)`)
- `runs.mirror.md`
- `runs/<runId>.triage-run.mirror.md`

Day pages (generated):
- `days/<YYYY-MM-DD>.day.mirror.md`
- `days/<YYYY-MM-DD>.triage-runs.mirror.md` (lists all runs for the day)
- `days/<YYYY-MM-DD>.buckets.mirror.md` (computed from latest run)
- `days/<YYYY-MM-DD>.next-actions.mirror.md` (computed from latest run)

Policy:
- **latest run wins** for `buckets` / `next-actions`.

#### `triage docc-site rebuild`

One-shot:

- `threads` then `views`

### `swift-threads-cli triage generate-mirror`

High-level primitive: produce a decision-grade run directory **and** update the DocC site.

Inputs:
- mirrors root
- routing map
- docc root
- runs root
- lane name + lane channel id

Outputs:
- writes a new `runs/<runId>/...` directory (canonical)
- calls `triage docc-site rebuild`

### `swift-threads-cli triage sync-to-discord`

Inventory primitive for missing mirrors.

Mode A (v1): expected inventory = all active threads under lane parent channel id(s).

Note: Swift CLI should write a deterministic plan artifact; OpenClaw executes the Discord fetch.

## Generated artifact naming

Use the filename stem as the DocC identifier.

- thread: `<threadId>.thread.mirror.md`
- triage-run page: `<runId>.triage-run.mirror.md`
- day page: `<YYYY-MM-DD>.day.mirror.md`

## Acceptance criteria

- One-command site regeneration:
  - `swift-threads-cli triage generate-mirror ...`
- Site pages show stable navigation without random/automatic linking:
  - hubs use `@AutomaticSeeAlso(disabled)`
  - threads hub uses `@TopicsVisualStyle(detailedGrid)`
- Every generated artifact is clearly labeled by suffix (`.mirror.md` / `.mirror.json`).
- Day bucket pages support multiple runs per day, with “latest wins” behavior.

## Open questions

- What is the authoritative prioritization algorithm for moving items into Now/Next/Blocked/Shipped beyond disk bucket + priority label?
- Should the triage run generator emit a canonical `render.mirror.md` that is posted verbatim to Discord, or should posting format remain separate?

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

## Incremental triage run authoring (single run dir, built up over time)

A triage run should be creatable **incrementally** (several CLI calls) while still producing a **single** disk-canonical run directory.

### Canonical directory

One run = one directory:

- `provisioned/threads/triage/runs/<runId>/`

Inside it, one schema per file:

- `triage-run.mirror.json` (manifest)
- `items.mirror.jsonl` (append-only snapshot log)
- `assignment.mirror.json` (overwrite-in-place)
- optional `render.mirror.md` (posting payload)

### Incremental CLI primitives

#### 1) Init

Create the run directory and manifest (idempotent, create-if-missing):

```bash
swift-threads-cli triage run init \
  --out-root /Users/sonoma/todo3/provisioned/threads/triage/runs \
  --run-id run-2026-02-19T05-00-00-0800 \
  --tz America/Los_Angeles \
  --lane-name "#pjm-todo3" \
  --lane-discord-channel-id 1470041553795023070 \
  --mirrors-root /Users/sonoma/todo3/.clia/profiles/operators/rismay/threads \
  --routing-map-path /Users/sonoma/todo3/provisioned/threads/thread-org-map.json \
  --docc-root /Users/sonoma/todo3/provisioned/threads/todo3.triage.docc
```

#### 2) Append item (repeatable)

Append one line to `items.mirror.jsonl`. To update an item, append a new line with the same `--id`; latest wins when rendering.

```bash
swift-threads-cli triage run append-item \
  --run-dir /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800 \
  --id discord.thread:1470893762019459122 \
  --kind thread \
  --title "Reminder system: appointments" \
  --bucket open \
  --priority p0 \
  --owners "rismay" \
  --next-action "Define reminder offsets and write the SOP" \
  --discord-url "<https://discord.com/channels/1407786883773104278/1470893762019459122>"
```

#### 3) Assign (overwrite-in-place)

```bash
swift-threads-cli triage run assign \
  --run-dir /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800 \
  --assigned-next-id discord.thread:1470893762019459122 \
  --assignees "rismay" \
  --why "Highest leverage unblocker today"
```

#### 4) Mirror (optional, repeatable)

Emit a canonical posting payload derived from `items.mirror.jsonl` + `assignment.mirror.json`:

```bash
swift-threads-cli triage run mirror \
  --run-dir /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800 \
  --out /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800/render.mirror.md
```

## Acceptance criteria

- One-command site regeneration:
  - `swift-threads-cli triage generate-mirror ...`
- Incremental run authoring works:
  - init → many append-item → assign → render produces one coherent run dir.
- Site pages show stable navigation without random/automatic linking:
  - hubs use `@AutomaticSeeAlso(disabled)`
  - threads hub uses `@TopicsVisualStyle(detailedGrid)`
- Every generated artifact is clearly labeled by suffix (`.mirror.md` / `.mirror.json`).
- Day bucket pages support multiple runs per day, with “latest wins” behavior.

## Open questions

- What is the authoritative prioritization algorithm for moving items into Now/Next/Blocked/Shipped beyond disk bucket + priority label?
- Should the triage run generator emit a canonical `render.mirror.md` that is posted verbatim to Discord, or should posting format remain separate?

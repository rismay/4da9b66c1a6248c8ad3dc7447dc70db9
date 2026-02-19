# Disk layout (canonical)

@Metadata {
  @PageKind(article)
  @TitleHeading("Disk layout")
}

## Topics

### Mirrors

- `.clia/profiles/operators/<operator>/threads/{open|incidents|blocked|closed}/<slug>/`
  - `*.summary.thread.clia.json`
  - optional `*.events.thread.clia.jsonl`

Buckets are canonical (`open|incidents|blocked|closed`). Priority is an attribute within a bucket.

### Runs

- `provisioned/threads/triage/runs/<runId>/`
  - `triage-run.mirror.json`
  - `items.mirror.jsonl`
  - `assignment.mirror.json`
  - optional `render.mirror.md`

Policy: multiple runs can exist per day; day bucket pages use **latest run wins**.

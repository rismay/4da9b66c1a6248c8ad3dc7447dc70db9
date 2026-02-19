# Incremental triage run (single run dir)

@Metadata {
  @PageKind(article)
  @TitleHeading("Incremental run")
}

A triage run should be creatable incrementally (several CLI calls) while still producing a **single** disk-canonical run directory.

## Topics

### Canonical directory

- `provisioned/threads/triage/runs/<runId>/`
  - `triage-run.mirror.json`
  - `items.mirror.jsonl` (append-only)
  - `assignment.mirror.json` (overwrite-in-place)
  - optional `render.mirror.md`

### Workflow

1) Init

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

2) Append item (repeatable)

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

To edit: append another line with the same `--id` (latest wins).

3) Assign

```bash
swift-threads-cli triage run assign \
  --run-dir /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800 \
  --assigned-next-id discord.thread:1470893762019459122 \
  --assignees "rismay" \
  --why "Highest leverage unblocker today"
```

4) Mirror posting payload (optional)

```bash
swift-threads-cli triage run mirror \
  --run-dir /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800 \
  --out /Users/sonoma/todo3/provisioned/threads/triage/runs/run-2026-02-19T05-00-00-0800/render.mirror.md
```

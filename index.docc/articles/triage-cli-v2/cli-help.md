# CLI help (proposed)

@Metadata {
  @PageKind(article)
  @TitleHeading("CLI help")
}

## Topics

### `swift-threads-cli triage --help`

```text
OVERVIEW: Triage tooling (mirrors → triage runs → DocC site).

USAGE: swift-threads-cli triage <subcommand>

SUBCOMMANDS:
  docc-site              Mirror DocC triage site pages/resources from disk.
  generate-mirror        Generate a decision-grade triage-run on disk, then mirror the DocC site.
  sync-to-discord        Plan + backfill missing on-disk thread mirrors from Discord (OpenClaw executes the plan).
  run                    Incrementally author a single triage-run directory.
```

### `swift-threads-cli triage run --help`

```text
OVERVIEW: Incrementally author a single triage-run directory.

USAGE: swift-threads-cli triage run <subcommand>

SUBCOMMANDS:
  init                   Create triage-run.mirror.json + items.mirror.jsonl.
  append-item            Append one snapshot line (latest wins per id).
  assign                 Overwrite assignment.mirror.json.
  mirror                 Mirror a posting payload markdown derived from items + assignment.
```

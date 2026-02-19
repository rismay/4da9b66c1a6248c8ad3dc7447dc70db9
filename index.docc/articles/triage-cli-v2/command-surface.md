# Command surface (v2)

@Metadata {
  @PageKind(article)
  @TitleHeading("Command surface")
}

All commands live under the `triage` tree. No compatibility shims.

## Topics

### DocC site mirroring

- `swift-threads-cli triage docc-site threads`
- `swift-threads-cli triage docc-site views`
- `swift-threads-cli triage docc-site rebuild`

UI rules:
- Chrome icon uses badge icons (`thread-icon-pX-sY`).
- Card image is the per-thread emoji pair SVG.

### Triage run generation

- `swift-threads-cli triage generate-mirror`
  - writes a new run dir (canonical)
  - mirrors the DocC site

### Discord backfill (plan)

- `swift-threads-cli triage sync-to-discord`
  - Mode A: expected inventory = all active threads under lane parent channel id(s)
  - Swift writes a deterministic plan; OpenClaw executes the Discord fetch

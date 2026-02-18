# Triage navigation rules

@Metadata {
  @PageKind(article)
  @PageColor(blue)
  @TitleHeading("Priority + severity grouping")
}

This site is an operational view of Discord threads, generated daily.

## Goals

- Keep the navigator predictable and fast for “what do I do next?”
- Bucket at **major** levels only (P0/P1/Other, S0/S1/Other)
- Still preserve **minor** rank (for sorting and per-thread labeling)

## Thread name conventions (inputs)

Threads may include machine-readable tokens in the name:

- **Priority:** `p<major>` or `p<major>.<minor>`
  - examples: `p0.1`, `p1`, `p1.7`
- **Severity:** `S<major>` or `S<major>.<minor>`
  - examples: `S0`, `S1`

(Other badges/emoji are allowed; these tokens are the ones we parse.)

## Buckets

We only bucket on the **major** value:

### Priority buckets

- **P0**
- **P1**
- **Other P** (P2, P3, … or missing priority)

### Severity buckets

- **S0**
- **S1**
- **Other S** (S2, … or missing severity)

## Sorting inside buckets

Inside each bucket, threads are sorted by rank:

- Primary sort: `major` (numeric)
- Secondary sort: `minor` (numeric; defaults to `0` if absent)

So within P1:

- `P1.1` sorts ahead of `P1.7`
- `P1` is treated as `P1.0`

## Display rules

- **Buckets never show minor rank** (no P1.7 sub-buckets).
- **Thread pages and list entries DO show minor rank** when present.
  - example list label: `P1.7 — …`

## Implementation notes

- DocC folder structure is for authoring organization only.
- Navigation structure is built via `## Topics` curation and **identifier-based** links (`<doc:...>`).

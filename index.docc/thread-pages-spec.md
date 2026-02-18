# Thread page DocC spec

@Metadata {
  @PageKind(article)
  @PageColor(purple)
  @TitleHeading("Stable thread pages + daily triage views")
}

This bundle is a **Discord thread registry** plus **daily triage views**.

## Goals

- **Stable pages**: one DocC page per thread (`thread-<threadId>`) that is overwritten in place.
- **Daily views**: regenerated pages that curate/sort the stable pages for today’s decision-making.
- **Mirrors as source of truth**: content comes from CLIA thread mirror summaries (not live Discord reads).

## File/identifier conventions

### Technology root

- `index.md` is the Technology Root (`@TechnologyRoot`).

### Stable thread pages (overwrite-in-place)

- Identifier: `thread-<threadId>`
- Path: `threads/thread-<threadId>.md`

Required content:
- H1 title format: `P<major>.<minor>: {Thread Title}`
- `@Metadata` with `@PageImage(purpose: icon, source: ...)`
- Metadata section including:
  - Owner (GitHub owner bucket)
  - Repo (repo bucket)
  - Priority (major+minor)
  - Severity (if present)
  - Discord thread URL
  - Updated/Created timestamps
  - Intent + DoD (when present)

### Daily triage views (regen)

- `latest.md` → curates today’s triage hub (`triage-sets-YYYY-MM-DD`)
- `archive.md` → curates prior `day-YYYY-MM-DD` pages
- `days/day-YYYY-MM-DD.md` → raw triage snapshot for that day
- `days/open-threads-YYYY-MM-DD.md` → view that groups stable thread pages by:
  - Priority buckets: P0 / P1 / Other P
  - Severity buckets: S0 / S1 / Other S
  - Lists show minor rank; buckets do not
- `triage-sets-YYYY-MM-DD.md` → daily operational hub
- `next-actions-YYYY-MM-DD.md` → daily next-action view (initially link-based)

## Bucketing/sorting rules

See: <doc:triage-navigation>

## Data sources

### Mirror summaries

Source:
- `.clia/profiles/operators/<operator>/threads/**/**.summary.thread.clia.json`

Key fields:
- `discord.threadId`, `discord.parentChannelId`, `discord.threadName`
- `priority` (`p1.7` etc.)
- `intent.why`, `intent.definitionOfDone`

### Routing map (owner/repo)

- `todo3/provisioned/triage/thread-org-map.json`

Maps `parentChannelId` → `{ owner, repo }`.

## Implementation

- Stable thread page generation: `todo3/provisioned/triage/bin/sync-thread-pages.mjs`
- Daily triage markdown + DocC view generation: `todo3/provisioned/triage/bin/generate-triage.mjs`

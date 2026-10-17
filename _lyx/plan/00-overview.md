---
format: 5
approved: true
language: go
---

# Plan: shedrecipe capabilities — segment settings and a told setting lookup

Stop hand-threading one `shedrecipe.Env` field per config value.
The per-row review maps fold into `Env.Segments`, one `SegmentSettings` per review segment, and `Env.Setting` gives a recipe row one told lookup it reaches by naming `module.key`.
The registry's `Constructor` learns its row's segment, `internal/configreg` supplies the hub-side lookup, `internal/loomrecipe` keys its budgets by segment, and the old maps are retired last, so every card builds and keeps the suite green.

## Card Index

1 — configreg-setting-lookup — add `configreg.SettingLookup`, the per-module snapshot lookup over the strict config stack
2 — constructor-row-identity — `Constructor` takes a `RowIdentity` carrying the row's name and segment
3 — shedrecipe-segment-settings — add `shedrecipe.SegmentSettings` and the `Env.Segments` and `Env.Setting` fields, legal nil in envkit's `nilLegal`
4 — envkit-template-setting — add `envkit.TemplateSetting` and fill `FullEnv`'s `Segments` and `Setting`
5 — wire-segments-and-setting — fill `Env.Segments` and `Env.Setting` from the loom and darn wiring
6 — review-rows-read-segments — the review entries read their segment and named settings, with build-time refusals, and the recipes name the settings
7 — budgets-by-segment — `New`, `NewDarn`, `Routing` and `DarnRouting` take budgets keyed by segment
8 — retire-row-maps — remove the unread per-row `Env` maps, `RowModelKeys` and their wiring helpers

## Shared Decisions

- `RowIdentity{Name, Segment}` is the row identity D2 names; it lives in `shedrecipe` beside `Constructor`.
- The review segment names stay the recipe's `segment:` strings; `internal/loomcli` holds the loom table and a `darnReviewSegment` constant, and no new shared constant is added.
- `Env.Segments` and `Env.Setting` join `envkit`'s `nilLegal`: their nil is legal for a recipe with no review segment or no named setting, so batten's wiring, which fills neither, needs no change.
- The old fields stay filled but unread between cards 5 and 8, so no card breaks a package it does not touch.

## PATTERN changes

- `PATTERN-recipe-setting-lookup` — adds: Giving a shed recipe row a per-row or per-segment config value: a plain value is named `module.key` and read through `Env.Setting`, a resolved one of a review segment's row is a `SegmentSettings` field; never its own `Env` field.

## verify:

```bash
go build ./...
lyx gate test ./internal/configreg ./cmd/lyx ./internal/shedrecipe ./internal/shedadapters ./internal/shedbuild ./internal/loomcli ./internal/loomrecipe ./internal/testkit/envkit ./internal/pattern ./internal/battencli ./internal/battenrecipe
lyx gate test --tags integration ./internal/configreg ./cmd/lyx ./internal/shedrecipe ./internal/shedadapters ./internal/shedbuild ./internal/loomcli ./internal/loomrecipe ./internal/testkit/envkit ./internal/pattern ./internal/battencli ./internal/battenrecipe
```

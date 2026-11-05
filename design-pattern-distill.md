# Distill PATTERN.md: cluster related entries, detail in background files

`pattern-distill` · Task · enhancement, pattern · running · Seed-Child

Depends on: [`raddle-recipe`](design-raddle-recipe.md)

## Seen (2026-11-05)

`PATTERN.md`'s overview sits at its headroom line (`internal/pattern/check.go`: `MaxOverviewBytes` 16 KB, `HeadroomBytes` 95%).
Every run that adds or widens an entry now stops at Plan-Write until the parent records trims of other entries: `pawl-orch-landing` needed two and `raddle-recipe` eight on 2026-11-05.

## Wanted

- The overview is distilled: entries that guard the same subject are grouped into one cluster entry, with one short rule line each, and every rule's detail lives in its background file.
- The section headings stay; a cluster lives under the section its subject belongs to.
- No rule is dropped: text that leaves the overview moves verbatim into the cluster's background file, and an entry folded into a cluster brings its own background file's text along.
- Every citation of a folded entry's ID, in code, tests, stencils, specs, skills, docs and heddle entries, is rewritten to the cluster's ID; a grep for each old ID finds none outside `docs/research` and history.
- The overview ends at or below about 60% of `MaxOverviewBytes`, so several runs can add entries before the next trim.
- `internal/pattern`'s checks pass on the result, and the cap and headroom values stay as they are.

## Bounds

- Runs in flight that declare PATTERN changes land first; this task waits for them through `depends_on`.

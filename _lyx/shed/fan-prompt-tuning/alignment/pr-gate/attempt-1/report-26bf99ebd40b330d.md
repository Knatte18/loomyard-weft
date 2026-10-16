---
verdict: APPROVED
---

# Alignment at PR-Gate — `fan-prompt-tuning`, PR 696 at `45dec3af9`

The diff of PR 696 against `main` builds what the board entry's goal and its two operator decisions of 2026-10-16 call for, as the decision record's D1-D10 shape it, and breaks no recorded decision.

## Against the goal and the decisions

- D2: `discussion_fan` and `plan_fan` become `FanList` (scalar or list), `discussion_fix_fan` and `plan_fix_fan` are added with `""` defaults in `internal/loomengine/template.yaml`, closed to the local layer, an empty list is refused, and every non-empty entry is resolved through `burlerengine.ResolveFan` with the entry's position in the refusal.
  The round usage record gains `fix_fan`, `fans_recorded`, `fix_lenses` and `fix_forks` in `internal/shedadapters/roundusage.go`, with `recordRoundFans` as the only writer of the names.
- D3: `fan_review` is retired through `RetiredKeys` and `retireFanReview` in `MigrateConfig`, with one reconcile line; no live code or doc names it outside that retirement and its tests.
- D4, D7: the fork reading, fanned and solo cluster rules, fixer self-check and fix-check rubric are new stencils under `contracts/stencils/burler/`, as `PATTERN-stencil-ownership` requires.
- D5, D6, D8: `review-discussion`, `review-plan`, `fix-discussion` and `fix-plan` and their lenses are in the embedded `internal/burlerengine/template.yaml`; the dormant fans stay.
- D7, D9: the twill fixer spawns with `ForkSubagents`, its engine refusal names the told fixer model key and fixer fan key, a fork violation fails the fix half, and a fork-count mismatch logs a warning.
- D10: the Bouncer's `exclude_lenses` answer reads the judged round's recorded fan and the next round's entry of `ClusterFans`, and degrades to no exclusion on an unreadable record.
- Refusal-spec rows for the widened and new refusals land in `contracts/specs/refusal-spec.md`; the module docs (`internal/burlerengine/doc.go`, `internal/shedadapters/doc.go`, `internal/burlercli/doc.go`, `internal/configcli/doc.go`) and `docs/overview.md` are updated.
- Out of scope held: the hub's own `loom.yaml` is untouched, as the record's operator decision leaves it to the orch after landing; Webster-Review and Merge-Review get no fan or fixer key; no `PATTERN.md` entry changes.

## Carried-over risks

- Discussion-Review closed at round 3 by an accepted hard-cap circling decision; the round-3 fixer report records F1-F6 fixed as proposed, and their resolutions are D2's archive-and-refuse path, the `fans_recorded` flag, the spawn-path-only write, the Bouncer degrade, `RunOpts.FixKey` and the `fix-affected-set` cap rule, each present in the diff.
- Plan-Review's carried MEDIUM `fixer-lens-list-sentence-overreach` was fixed in the round-3 fixer report.
- The record's open risks (unbenched lens texts and fork caps, fork overlap, lens-name shadowing, `codex-engine`'s stale list of fork-free roles, `loom-model-key-names` starting from the new key set) are follow-up measurement or other entries' work and call for no change here.

## Overlap

`lyx loom peers` lists no peer run and no parent commit (`own_row` 22), so no overlap and no landing order.
Board entries that touch this task, none made obsolete by it:

- `loom-model-key-names` lists `fan_review` among the keys to rename; the record's open risk says it must start from this run's key set.
- `codex-engine` lists the Burler fixer as fork-free; after this run only a `single` fixer is.
- `plan-writer-finding-classes` runs its audit after this task lands, by its own body.
- `review-inline-code` keeps the reviewer code pack, which the record's Out list leaves to it.

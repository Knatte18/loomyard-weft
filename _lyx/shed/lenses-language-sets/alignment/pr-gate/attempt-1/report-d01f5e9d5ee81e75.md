---
verdict: APPROVED
---

## Alignment at pr-gate: lenses-language-sets, PR 817 at 932ae4e43

The diff against `main` (merge base `684c04305`, equal to `main`'s head) carries the board entry's goal and the operator decision of 2026-10-22: a shared language-neutral base, a predefined language layer per language, and a per-repository `remove`.

- D1: `internal/lenses/template.yaml` keeps every lens and fan name. It rewords `error-handling`, `test-gaps`, `performance`, `api-design` and `concurrency` neutrally, and rewords `pattern-fit`, `code-pattern-fit` and `docs-consistency` to read PATTERN where present, otherwise the `CLAUDE.md` rules.
- D2: `internal/lenses/sets/go.yaml` carries the five lenses' previous prose verbatim.
  `csharp.yaml` adds `numerics` and overrides `review-code` and `full` as the base fan followed by `numerics`, and its `error-handling` and `numerics` prose defer to the repository's rules on guards.
  `python.yaml` is present.
- D3 and its refinement: `lens_set` is a Guard key read from the base layer only, and a hub worktree file holding a differing value is refused (`internal/configengine/overlay.go`).
- D4: `configengine.Overlay` and the one exported composer `EffectiveTemplate`/`TemplateUnder` are used in load, `Set`, `Edit` and configsync.
  `configreg.Fingerprint` covers the overlay documents.
- D5: `remove` is a Guard key applied only in `lenses.Load`.
  `EmptiedFans` drives the `ResolveFan` refusal, and the warning for skipped entries goes through `configengine.WarnSkippedEntries`, so lenses gains no import.
- D6: no code writes `lens_set` for a hub.
- PATTERN changes: the `PATTERN-template-overlay` line is added, and the clauses of `PATTERN-sparse-hub-base` and `PATTERN-config-layers` are shortened, each as recorded after Discussion.
  The overview intro line was also cut; the plan's `pattern-overview-headroom` decision records that cut.
- Docs: `internal/lenses/doc.go`, `internal/configengine/doc.go`, `internal/configcli/doc.go`, configsync and the `docs/overview.md` module-table line.

The open risks carried into the record are design and test details for the review segments, and none is a scope deviation.
The emptied-fan risk that `remove` creates is recorded as accepted.
The carried Plan-Review finding `lensesfixture-leaks-new-config-fields` is handled in the diff: a separate `burlerFixture` in `internal/configsync/configsync_test.go` marshals only `lenses` and `fans`.

No change the goal does not call for was found, no goal point is left out, and no recorded decision is broken.

## Overlaps

`lyx loom peers` lists no parent commits (`own_row` 21).

- `machine-handover` (row 11): both change `docs/overview.md`, each its own module-table line, which is shared bookkeeping, a text-only overlap.
  Its `pattern/PATTERN-leaf-packages.md` change touches no file this run changes.
  No design conflict.
  Landing order: this run (row 21) lands first, and `machine-handover` takes these changes at Merge-In.
- `quarry-csharp` (row 11): no shared file; its decision record puts `lenses-language-sets` out of its scope.
  No overlap.
- `tender-pause-holds-reprompt` (row 7): no changed paths, and its decision record touches no lenses or configengine behaviour.
  No overlap.

## Superseded

No board entry is made obsolete by this run.

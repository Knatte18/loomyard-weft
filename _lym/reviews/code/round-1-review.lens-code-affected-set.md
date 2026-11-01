```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (code-affected-set lens): per change, the set found and whether it is covered**

- **`pauserecord.HoldCaller` and `Coverage.Held`.**
  - Set: `pauserecord/doc.go` (updated), `dobbyengine` `RunView.Held`, `planPass`, the owner-pause recheck in `passes.go`, `hibernateengine/wake.go` and `tendercli/deps.go` (both judge only `OwnerCaller`), and `beamcli/dobbydeps.go:321`.
  - Result: covered. The other `OwnerCaller` readers are untouched on purpose, and the hold is never confused with an owner entry.
- **Darn second stop marker (`ConditionStopMarkerPath`).**
  - Set: `darnengine/spec.go`, `loomshed/darnwrite.go`, `loomcli/darnwiring.go`, `shedrecipe/entries_darnwrite.go`, `testkit/envkit/envkit.go`, the stencil, and the tests for each.
  - Result: covered. A grep for `stop_marker_path|StopMarkerPath` finds no consumer outside this set, and the stencil header comment names the new marker.
- **Board payload keys `front` and `on_land`.**
  - Set: `UpsertPayloadKeys`, the three payload write paths in `store.go` (upsert, batch, merge), `Rename`, and the `boardcli` upsert `Use` and `Long` help.
  - Result: covered. A grep for `notify_on_land` (the sibling key) shows no spec or markdown page that lists payload keys outside these files.
- **`adopt` behaviour and docs.**
  - Set: `docs/overview.md` (the adopt, beam status and batten paragraphs), `adoptcli/doc.go`, `beamcli/doc.go`, and `pattern/PATTERN-test-speed.md`.
  - Result: covered.
- **Dobby front-task starts (`startReady`, `BuildView`, `FrontFirst`, `SettleFronts`).**
  - Set: dobby `doc.go`, the battencli overview readers, and `beamcli/dobbydeps.go`.
  - Result: covered. The new `Front` and `Project` fields on `TaskView` and `Project` on `ViewInput` are supplied by `beamcli`, and the diff also touches the overview readers.
- **PATTERN landing.**
  - `PATTERN.md` carries both declared entries with the declared text.
  - Result: covered.
- **Commit messages.**
  - The range's commit messages carry no `Pattern-Departure:` trailer.
  - Result: nothing to judge.

**Findings**

1. **LOW / consistency — `docs/overview.md` omits the new front-task piece and its dobby behaviour.**
   - `docs/overview.md:447` mentions "a front task" only in passing; the overview line is `docs/overview.md:458`.
   - No overview entry describes `internal/fronttask` or the dobby behaviour the diff adds: an open front task is started first and past `max_runs`, other starts wait, and the `on_land` action runs from the closed table.
   - A grep of the non-`_lym` markdown finds `fronttask` only in `PATTERN.md`.
   - The record says "package docs of adopt, dobby and board follow", and those package docs are updated, so this is a gap in the repository-wide overview rather than a missed required doc.
   - Fix: add a short entry to the dobby/beam section of `docs/overview.md` naming `internal/fronttask` and the start-ahead behaviour.
2. **LOW / consistency — the `lym board upsert` help text was shortened in lines unrelated to this task.**
   - `internal/boardcli/cli.go` near lines 501–516: the commit "fix: keep the adopt and board upsert help pages within the Long cap" cut the `created`, `revised`, `profile` and `pull_request` descriptions and the `--body-file` text.
   - The cut drops the "refused when the payload is itself `-`" clause and the "laid under run_settings" and "beam.yaml's pull_request mode" detail.
   - These edits have no card or decision behind them, but they are forced by the Long cap, so any harm is small.
   - Fix: confirm the dropped `--body-file` refusal clause is still accurate to say nothing about, or restore it by shortening other lines.

Beyond that, I found no affected member the diff leaves wrong.

**Tool calls (3 used)**
1. A grep for `ConditionStopMarkerPath`, `notify_on_land` and `HoldCaller` consumers, plus the diff of `PATTERN.md`, `docs/overview.md` and `boardcli/cli.go`: the manifest lacked these.
2. A grep for `stop_marker_path`, adopt mentions in the markdown, and adopt in `tender` and `batten`: the manifest lacked the other consumers of the stop marker.
3. A grep for `fronttask` in `docs/overview.md` and the markdown, plus a listing of `pattern/`: to check whether the front-task piece is documented.

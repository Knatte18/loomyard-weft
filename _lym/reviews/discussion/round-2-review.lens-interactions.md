```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens over the decision record's decisions.

**Pairs checked**
- D2 × D3, build order (build, then build function, then pack, then fingerprint): held.
- D2 × D3, the board-HEAD rule against the per-copy amend and `push --force`: **broken** (finding 1).
- D2 × D3, CopyHub without the walk still failing on a poisoned template: held.
- D3 × D7, the fold amending the tip a builder committed, with the reflog criterion: held.
- D3 × D7, "a copy's board stays clean" against loomcli `beamHub`'s build: **broken** (finding 2).
- D3 × D1, `pack-refs --all` against the reader's loose-then-`packed-refs` lookup: held. The packed-refs criterion covers it, and a ref a test updates later is read as a loose ref first.
- D3 × D5, `newRecordsFixture` on a packed copy that carries the amend reflog: held. My grep found no production read of a ref file, and the template build also runs `forgetCloneCorrespondence`.
- D3 × D6, three `TestAnchorChange` refusal rows on one CopyHub with no walk: held. D3's Bound covers only `SharedHub`, and the rows refuse before any write.
- D6 × D7 task 8, the `TestVerbCases_CleanState` checks in `runCell` against the arrange split into build and derivation: held. The checks run on the copy after the derivation.
- D6 × D7 task 7, the shared CopyHub of the refusal rows against `newAnchorHub` moving onto a registration: held. D9 orders task 6 first.
- D7 × D2, steps that stay in the test (TempDir, process-wide calls, HEAD not at its upstream): held, except `endedHub` (finding 1).
- D7 × D2, `newBadReedUpFixture`'s `SeedConfig` commit on the prime records, which is not the board: held.
- D7 × D2, battencli `beamHub` with `SkipPush`: held. `Add` writes no board commit; the beam mark is an origin commit in the pair's records.
- D7 hubreconcile × D5 × D8, all edits in `ensure_integration_test.go` in one task: held.
- D4 × D1, config written from Go against the reader's reads of `core.ignorecase`, `core.bare` and `extensions.refStorage`: held. The keys D4 writes do not overlap them.
- D9 task 1 against tasks 2 to 11, and tasks 4 against 2 and 3: disjoint files, held.
- D9 tasks 5 against 6 to 8: `replay_integration_test.go` belongs to task 5 alone and `endedHub` sits in `originreconcileended_integration_test.go`: held.
- D10 against `gate-tier-binaries-only`: Open risks names it, held.

**Findings**

1. **BLOCKING · design · D2 last bullet, D3 fold, D7 `internal/fabricengine` anchor and origin (`endedHub`)**
   D2 promises that D3's force-push "never drops a commit the builder made". `endedHub` breaks that promise.
   - D2 checks "the board's HEAD equal to its upstream tip", and nothing in the record says whether "upstream tip" means the tracking ref or the records bare's branch.
   - `endedHub` runs `publishBoard`, which commits and pushes so the tracking ref equals HEAD. It then runs `originBoard`, which commits the "done" board straight into `hub.RecordsBare` on that branch. That leaves the bare ahead while HEAD still equals `@{u}`.
   - If the guard compares HEAD with `@{u}`, the build passes. D3's per-copy `push --force origin HEAD` then rewinds the copy's bare and drops origin's "done" commit, so every ended test sees no run ended on origin.
   - D7's rule that keeps such a step in the test has the same ambiguity, so it does not keep `originBoard` out of the build.
   - Fix: D2 states that the build compares the board's HEAD with the records bare's own branch, not the tracking ref. D7 names `originBoard`, and any commit made straight into a bare, as a step that stays in the test.
   - Evidence: `internal/fabricengine/originreconcileended_integration_test.go:58-71`, `:87-97`; `internal/hubforge/template.go:182-188`.

2. **LOW · consistency · D3 Bound ("A copy's board stays clean against its HEAD and upstream"), D3's first acceptance criterion, D7 `internal/loomcli`**
   D3 claims every copy's board is clean, but loomcli `beamHub` is listed in D7 to move onto a template, and its build writes `board.json` and the shortname file into the board without committing them.
   - D2 checks only HEAD against upstream, so that build passes and every copy's board is dirty.
   - The claim therefore holds only for shapes and for builds that commit what they write.
   - Fix: limit D3's claim and its criterion to what the build left, or have D2 also refuse a build that leaves the board dirty. The second choice would keep loomcli's `board.json` write in the test.
   - Evidence: `internal/loomcli/projectrecord_integration_test.go` `beamHub` (the `os.WriteFile` calls into `hub.BoardDir()`); `internal/hubforge/template.go:155`.

Tool calls:
- grep of `publishBoard` and `originBoard` in `internal/fabricengine` tests: the manifest read `endedHub` but not the helpers that write its board on the local side and on origin.

```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change in the diff I traced who calls, reads, asserts or documents it, and checked whether the diff leaves any of them wrong.

**Per change: what I found, and whether it is covered**

1. **`newCommitStatusSeam` adds a second core with a no-op Push and no admission, owed-retry or sleep deps** (`internal/battencli/commitstatus.go:237-247,264-266`).
   - Caller `battenCommitStatusSeam` → `wire.go:1720`: covered. The signature is unchanged.
   - Its own file header (`commitstatus.go:6-10`) and doc comment (`:209-215`): covered, both updated.
   - `internal/statuscommit` contract for `Deps.Push` and `New` (`statuscommit.go:32-34`, package doc `:1-3`): not covered. See finding 2.
   - `TestNewCommitStatusSeam`: covered, with new rows.
   - The loom seam (`loomcli/wiring.go:344`): untouched, and nothing there needs a change.
2. **Batten transition behaviour: the start rows commit without pushing.**
   - Rule document `PATTERN.md` entry `PATTERN-run-fsm`: covered.
   - Background `pattern/PATTERN-run-fsm.md` (`:4`, new section `:38-47`): covered.
   - Background `pattern/PATTERN-pair-remote.md` body (`:20`): covered.
   - The `PATTERN-pair-remote` entry line in `PATTERN.md` and its background opening (`pattern/PATTERN-pair-remote.md:3`): not covered. See finding 1.
   - Readers of origin's batten record during the window: `fabricengine.OriginBattenStatus` → `pairteardown.HandOver` / `HandOverOwnedElsewhere` (`handover.go:111,164`). These read it only to compare intents, so nothing goes wrong there.
   - Adopt's `takeOverRuns` / `movedRun` reads the takeover entry, which this change does not affect.
   - `dobbyengine.CatchUp` (`Ahead` → `PushOwed`) pushes the unpushed start commits after an absence. The summary's claim holds.
   - `docs/overview.md:339,1882-1885`: makes no claim about per-transition pushes that this diff falsifies.
   - `docs/research/...`: dated research records, exempt.
3. **Test struct gains `wantPushes`, and the existing rows set it equal to `wantCommits`**: covered. The assertions on the existing rows are as strict as at the base.
4. **Commit message and `Pattern-Departure` / `Loosened-Check` / `Root-Cause` trailers**:
   - No trailers are present.
   - The message's claims match the diff: the resumed or halted start row still pushes, every row after Run-Shed pushes, and a takeover meets the next push.
   - The 40–70 s and 1–13 s figures describe the problem, not a claimed improvement.
   - The directive declares no PATTERN changes, so nothing is owed under "Declared PATTERN changes landed".

**Findings**

1. **LOW · consistency** — `PATTERN.md` entry `PATTERN-pair-remote` (line 28) and `pattern/PATTERN-pair-remote.md:3`.
   - Both still state "the code branch goes along after the records side, both before the run acts", with no exception.
   - The diff adds the exception for the uninterrupted start to `PATTERN-run-fsm`'s entry and opening, and only to the pair-remote background's body (`pattern/PATTERN-pair-remote.md:20`).
   - So the pair-remote rule headline now contradicts what the code does at the start rows.
   - Evidence: `PATTERN.md:28` (read in step 1's constraints listing); `pattern/PATTERN-pair-remote.md:3` versus `:20`.
2. **LOW · consistency** — `internal/battencli/commitstatus.go:238-242` against `internal/statuscommit/statuscommit.go:32-34`.
   - `Deps.Push` documents that it is required and that "New panics on a nil Push, since a seam that pushes nothing leaves its transitions unpublished".
   - The new `startCore` passes `func() error { return nil }` to get past that guard.
   - That builds the very seam the contract forbids, and the contract text was not changed. The package doc's "pushes it synchronously, so the transition's records are on origin before the run's next producer starts" is likewise contradicted for this caller.
   - Either the statuscommit contract states that a pushless seam is allowed, or batten commits the start rows without building a core whose push does nothing.
   - Evidence: `statuscommit.go:1-3,32-34,79-82`; `commitstatus.go:237-242`.

Tool calls:
1. grep for the "pushes before the run acts" wording across the repo, to find rule documents outside the manifest.
2. grep listing docs that mention the status commit or a transition push, to find docs the manifest lacked.
3. grep of `PATTERN-daemon-wakeups.md`, `docs/overview.md`, the `battencli` doc and `runfsm` / `pairteardown` docs for transition-push claims, to check each hit.
4. grep of `docs/overview.md` for every-transition push claims, plus `statuscommit.New` test callers, to look for stale claims and other users of the core.
5. grep for readers of origin's batten record (`OriginBattenStatus`) and for tests asserting `-> running` commits, since those consumers were not in the manifest.

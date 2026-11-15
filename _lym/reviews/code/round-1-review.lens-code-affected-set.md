```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the diff's changes force outside the lines it touches. Per change: set found, covered or not.

1. `battencli.ClearDoneRun` and `clearDoneRun(admits)`, replacing `clearMotifRun`, plus `restoreMotifRunRecords` renamed to `restoreDoneRunRecords`.
   - Set found: `ClearMotifRun` (`internal/battencli/cleanup.go:118`), its callers `startguard.go:150` and `beamcli/deps.go:72`, and the test file `cleanup_test.go`.
   - Covered: no reference to the old lowercase names is left (grep shows none outside the diff). The tests were renamed.

2. `Deps.ClearDoneRun` seam, filled at `beamcli/dobbydeps.go:177`.
   - Set found: the only production `dobbyengine.Deps{}` literal is `hubDobbyDeps`, and it is filled there for every dobby. The test literals (`dobbytrace_integration_test.go`, `dobbydeps_integration_test.go`) only return errors or read `RunNotices`.
   - Covered: no other construction needs the seam. A nil seam clears nothing.

3. The dobby tick now clears done runs after counting landings (`loop.go:697`, `tidy.go:117`). No other writer or reader of `d.landed` exists, and `CountedLandings` is only read and written in `tidy.go` and `donerun.go`.
   - Covered.

4. Docs that describe done-run behavior.
   - `internal/dobbyengine/doc.go` (the done-run clear section, and the line at 165) and the `cleanup.go` and `cli.go` help text are updated.
   - `internal/beamcli/doc.go:59` says "Any other done run stays, and the first pass refuses it naming `lym batten cleanup --apply`". The diff leaves it unchanged.
   - `docs/overview.md:816` says "any other done run is still refused, naming `lym batten cleanup --apply`". The diff leaves it unchanged.
   - `docs/overview.md:1617` describes `lym batten cleanup` and the motif-run clear. It does not mention the dobby's clear.
   - Not covered. See F1.

5. PATTERN entries.
   - `PATTERN-dobby-runs` and `PATTERN-owner-machine` constrain who starts, resumes, pushes or lands a run. The new clear runs inside the parent's dobby and calls the existing records-commit and push-spawn path.
   - No declared PATTERN changes exist, and the diff has no `Pattern-Departure:` trailer, so none is needed.
   - Covered.

6. Commit message claims.
   - It claims the clear happens "once its landing is counted and its landing notice decided", and the code matches.
   - It claims records and scratch only, and the diff removes no task branch. This is the focus directive's scope question and belongs to the handler. The lens finds no affected-set member for it.

**F1 — MEDIUM, consistency.**
- Location: `internal/beamcli/doc.go:59` and `docs/overview.md:816`.
- Summary: both say a done run other than a motif run stays and refuses a later start, naming `lym batten cleanup --apply`. The dobby now removes every done run once its landing is counted and its notice decided, so that text is stale. `docs/overview.md:1617` also omits the dobby's clear from the `lym batten cleanup` description.
- Fix: update `beamcli/doc.go:59` and `overview.md:816` to say a done run is normally removed by the parent's dobby. The refusal naming `cleanup --apply` then applies only while the dobby has not yet cleared it. Add the dobby's clear to `overview.md:1617`.

**F2 — LOW, consistency.**
- Location: `internal/battencli/cli.go` help text (the diff hunk near line 314), against `internal/dobbyengine/doc.go:487`.
- Summary: the help says the dobby removes a done run's record "once it has counted its landing and decided its landing notice". The doc adds further preconditions: no pass in flight, driver lock free, and no board-tidy agent in flight. The help is a simplified claim, and the `cleanup.go` help text carries the same wording.
- Fix: no change needed beyond optionally noting "while it can" in the help text. Raise it only if the help is meant to be exact.

Tool calls:
1. grep for `ClearDoneRun`, done-run wording and `Deps{}` literals, plus `PATTERN-dobby-runs.md`. My context lacked the whole-repo callers and docs.
2. `docs/overview.md` lines 1610–1622 and 810–818, and `beamcli/doc.go` lines 50–58. My context lacked these doc passages.
3. grep for "done record" and "done run" in the docs. I needed to find any other stale passages.

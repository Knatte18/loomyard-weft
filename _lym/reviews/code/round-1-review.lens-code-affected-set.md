```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I listed what reads or describes it and checked whether the diff left any of it wrong.

**Per change, the affected set and whether it is covered:**

1. **`Runner.StopStepRuns` now runs through `stopStepRunsWhere`, and probes reed only once a run qualifies.**
   - Callers: `internal/loomcli/looparm.go:157`, reached from `watchdogKillNote` and `orphanedStepNote` (`arm.go:444-445`). Unaffected.
   - Tests: `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns`, `enginedispatch_test.go:739` (still stops one run and removes `strand-1`) and `looparm_test.go:113` (a "probe reed" error rendered into the note). All still valid.
   - Docs: the `StopStepRuns` godoc gains the probe line, and `shuttleengine/doc.go:409-413` is updated.
   - `completionsignal_enforcement_test.go` lists no `StopStepRuns` site. The gate already judges whether the moved `Errorf`s count.
   - Covered.
2. **New `Runner.StopOrphanedStepRuns`.**
   - Its one caller is `loomCLI.stopOrphanedStepRuns` (`looparm.go:169`).
   - Docs: the `stepstop.go` file header and godoc, and `shuttleengine/doc.go:409`.
   - Covered.
3. **`LoopSpec.ReconcileOrphanedSteps` replaces `reconcileInterruptedStep` and `newestInterruptedTraceID`.**
   - A repo-wide search of `.go`, `.md`, `.tmpl` and `.yaml` finds no reference left to either removed symbol, nor to the "reconciled the interrupted step" mark.
   - `OnStepOrphaned`'s godoc drops the loop-start call.
   - `shedverbs/doc.go:198-207` is rewritten, and `loopwait.go`'s record comment is updated.
   - Tests: `loop_test.go` and `loopwait_test.go` are updated.
   - Covered, except finding 1 below.
4. **The loop-log mark text changes to "reconciled the steps whose process is gone: …".**
   - Its only reader is `loop_test.go:379`, which the diff updated. No docs or fixtures assert it.
   - Covered.
5. **loomcli wiring.**
   - `stepRunsNote` is split into `stepStopNote` and `stepStopRunner`, and `orphanedStepsNote` and the `stopOrphanedRuns` seam are new.
   - The callers are `watchdogKillNote` and `orphanedStepNote` (`looparm.go:131,138`) and `arm.go:446`.
   - Tests: `looparm_test.go` (note rendering) and `cli_test.go`, which is updated.
   - The shared renderer's error wording now also serves the multi-step note: see finding 2.
6. **Rules, commits and patterns.**
   - The range's commits carry no `Pattern-Departure:` trailer.
   - The directive declares no PATTERN changes.
   - No stencil is touched.
   - Nothing to check.

**Findings:**

1. **LOW · consistency** · `internal/shedverbs/doc.go:206`
   - Summary: the doc says a step that ended in an error "then does not leave its runs to the re-step".
   - The same doc uses "re-step" for the loop's one immediate transient re-step (`doc.go:85`, `loop.go:195`).
   - That in-loop re-step runs with no reconcile, and no `OnStepOrphaned` call, because an envelope exists. A transient-error step's runs do reach it.
   - Evidence:
     - `shedverbs/doc.go:85` and `doc.go:206`
     - `loop.go:243-247`: reconcile runs only before `stepUntilStop`.
     - `loop.go:453`: the transient re-step inside `stepUntilStop`.
   - Fix: say "the next loop's first step" or "a re-run of the step", not "the re-step".
2. **NIT · consistency** · `internal/loomcli/looparm.go:121-123`
   - Summary: `stepStopNote` renders a stop error as "could not stop the step's runs: …".
   - That wording now also reaches the reconcile note, which covers the runs of every gone step, with no one step meant.
   - The loop prefixes that note with "reconciled the steps whose process is gone: …" (`loop.go:245`). A failed probe then logs "reconciled the steps whose process is gone: could not stop the step's runs: probe reed: …".
   - Evidence: `looparm.go:121-123`, `looparm.go:144-145` and `loop.go:244-245`.

**Tool calls:**
1. grep for stale references to the removed reconcile, plus the `enginedispatch_test.go` StopStepRuns rows: the manifest held no repo-wide search of docs, stencils and fixtures.
2. grep of the package docs, PATTERN files and specs for the loop hooks and the reed-probe test text: needed to find doc and test readers outside the manifest.
3. Read of `stepStopNote`'s body and the completion-signal enforcement list: the manifest held only the diff hunk of the note renderer.
4. grep of the `shedverbs` docs for "re-step" and "interrupted record" wording: needed to confirm the docs the diff touches agree with the rest of the package doc.

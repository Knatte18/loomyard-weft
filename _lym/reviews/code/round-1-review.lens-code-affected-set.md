```yaml
lens: code-affected-set
outcome: reported
```

**Per change — affected set, covered or not**

1. `driveNotices` / `driveHandBack.Judged` / `goDriver.judged` (watched hand-back of a judged blocked stop now sends the run-id line):
   - `sheddriver/doc.go:65-70` — updated, covered.
   - `drivehandback.go` doc comment — updated, covered.
   - `handBackRun` caller — unchanged, consistent, covered.
   - `battenshed/doc.go:65`, `notice.go` hold docs — describe batten's own hold (unchanged); no edit needed.
   - Notify seams `loomcli/drivewiring.go:267`, `battencli/drive.go:292` — route any line, no shape change; covered.
   - Tests `TestDriveNotices`, `TestGoDriver_Settle` — new rows; covered.
2. `ValidatePlan` gains `overview-frame-edited` check:
   - `loomshed/doc.go`, `ValidatePlan` doc — updated, covered.
   - `TestNewPlanGate` rows — covered.
   - `loomcli/parity_test.go` (gate/verb parity) — both sides share `ValidatePlan`; covered.
   - `contracts/specs/loom-plan-checks-spec.md` — **not updated** (finding 1).
   - `PATTERN-refusal-way-forward` — not applicable: `refusal-spec.md:38` exempts a validation verb's findings envelope ("a verdict, not a refusal").
3. `fabricengine.CommitWithTrailers` / `commit` / `commitWeftLocked` trailers param:
   - Callers `commitBothSides`, `export_test.go` `CommitWeftForTest` — updated, covered.
   - `fabricengine/doc.go` — updated, covered.
   - `TransitionTrailerKey` doc — updated to name webster's reset; covered.
   - Readers of the trailer (`handover.go` `lastTransitionCommit`, `codetipsettle.go` `ReturnToPreviousTransition`) — read whatever carries it; consistent.
4. `fabricSync` variadic trailers + `websterCLI.runID`:
   - All other `fabricSync` callers (beginbatch, rebaseline, run, restoreplan, fixplan, fixverify, recordbatch, recoverbatch, acceptaudit) — pass none; behaviour unchanged; covered.
   - `sync.go` file header — updated; covered.
   - `webstercli/doc.go`, `reset.go` header — updated; covered.
   - `wireHub` / `wireStandalone` set `runID`; reset test fixture sets `runID` — covered.
   - `pattern/PATTERN-run-fsm.md` (what a transition commit is and how it pushes) — **not updated** (finding 2).
5. Commit messages / departures: no `Pattern-Departure:` or `Loosened-Check:` trailers; directive declares no PATTERN changes, so nothing to check for landing.

**Findings**

1. **MEDIUM — consistency** — `contracts/specs/loom-plan-checks-spec.md` (section "Run state and parts")
   Summary: The pinned plan-checks spec lists the loomshed context check `done-card-edited` with what raises it and its way forward; the diff adds the sibling blocking check `overview-frame-edited` to `ValidatePlan` but no spec entry for it.
   Evidence: `contracts/specs/loom-plan-checks-spec.md:3` ("Contract — pinned… an entry here says what raises the check"), `:287-290` (the `done-card-edited` entry); new check at `internal/loomshed/gates.go:388-411` in the diff.

2. **MEDIUM — consistency** — `internal/webstercli/reset.go:177-183` / `pattern/PATTERN-run-fsm.md` ("A transition pushes before the run acts")
   Summary: The reset's records commit now carries `Run-Transition`, so `lastTransitionCommit`/`ReturnToPreviousTransition` treat it as a run transition, but the pattern background still says a transition takes gate admission through `statuscommit` and pushes records then code before the run acts; the reset's commit goes through `fabricSync`/`Fabric.Commit`'s detached push outside `statuscommit`, and no `Pattern-Departure:` trailer or background update records this writer.
   The handler should judge whether this breaks the listed `PATTERN-run-fsm` constraint (BLOCKING) or only leaves the background stale.
   Evidence: `pattern/PATTERN-run-fsm.md:21-24`; `internal/fabricengine/handover.go:350-360`; `internal/fabricengine/trailer.go:17-22`; `internal/webstercli/sync.go` (`f.CommitWithTrailers`).

**Tool calls**
1. Grep for `done-card-edited`, `Run-Transition`, `webster reset`, and the run-id line wording across docs/specs/stencils — context lacked the docs/specs that describe the changed behaviours.
2. Read `loom-plan-checks-spec.md` entry, parity/overlay test `done-card-edited` lines, stencil/webster-template lines — context lacked those affected-set members.
3. Read plan-checks spec intro, `PATTERN-refusal-way-forward` background, and package-doc `Run-Transition` mentions — needed to judge whether the spec claims to cover every check and whether the refusal rule applies.
4. Read `gates.go` `done-card-edited` finding detail and `refusal-spec.md` exemptions — needed to decide whether a plan-gate finding counts as a refusal under the pattern.

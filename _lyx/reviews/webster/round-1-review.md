---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/fabricengine/mergestage.go:165
    summary: MergeUntrackedFiles also lists the weft's untracked files, so lyx's own uncommitted weft state halts a resolved conflict as stuck, or errors it when the path sits outside the visible tree, although the weft is no merge participant and card 4 scopes the list to the task worktree
  - id: F2
    severity: LOW
    location: internal/verifytree/verify.go:144
    summary: Verify reads HEAD's tree once before the run and writes it into the record after a pass without re-reading it, so a commit landing mid-run records a pass for a tree the command did not run on
  - id: F3
    severity: LOW
    location: internal/verifytree/verify.go:96
    summary: DirtyPaths parses porcelain v1 text, so a quoted path keeps its C escapes and an unquoted name containing ' -> ' is cut as if it were a rename, and those mangled paths reach the Publish, Finalize and gate findings verbatim
  - id: F4
    severity: LOW
    location: internal/friction/friction.go:35
    summary: Prose outside card 17's edit list still names the retired integration fork, integration report and ParseIntegrationReport
  - id: F5
    severity: NIT
    location: internal/websterengine/verifygate.go:107
    summary: New and changed Go comments join independent clauses with a semicolon, or put two sentences on one line, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F6
    severity: NIT
    location: internal/websterengine/config.go:81
    summary: The config comment says verify_gate_attempts at 0 leaves the verify gate no attempt, but shuttle turns a 0-attempt entry off, so the must-pass verify would silently not run
---

Range: `_lyx/shed/verify-publish-hardening/status.json` names `product.parent: main`, and no `_lyx/loom/rework/` round exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (`5805c58fb..efdf79946`, cards 1–20).
`go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass on HEAD.

### [MEDIUM] MergeUntrackedFiles reaches into the weft

**Location:** `internal/fabricengine/mergestage.go:165` (`MergeUntrackedFiles`), consumed by `internal/mergeresolve/mergeresolve.go:125`.

**Issue:** Card 4 defines `MergeUntrackedFiles` as the untracked, non-ignored paths "in the task worktree", with junctioned `_lyx` and `.lyx` excluded.
The implementation also lists `f.weft.UntrackedFiles()` and unifies them through `unifyConflictPaths`.
`MergeIn` merges the warp checkout alone (`merge.go:113`: "the weft side is not a merge participant"), and `pairDirtyReason` deliberately ignores the weft because checking it "could only refuse a merge that would have been right".
A weft untracked file can never enter the warp merge commit, so listing it does not protect the merge.
It does cause harm: `resolveConflicts` aborts a fully resolved merge as stuck whenever lyx has any uncommitted new file in the weft at that moment, and a weft untracked path outside `<anchorRel>/_lyx` makes `unifyConflictPaths` report unmappable, so `Resolve` returns an error instead of concluding.

**Fix:** List the warp side only, as card 4 states, and drop the weft read and the `lyxcwd`/geometry resolution it needed.
Update the doc comment to say the weft is not a merge participant.

### [LOW] The record can name a tree the command did not run on

**Location:** `internal/verifytree/verify.go:144` (`Verify`).

**Issue:** `headTree` is read once, before the run, and that value is written into the record after exit 0.
A commit that lands while the command runs leaves the record claiming a pass for the pre-run tree, although the command read some of the new tree's files.
The focus asked to settle this.
The record is only wrong in that narrow window: a command that dirties the worktree leaves the record naming the committed tree the run started from, and the next call refuses the dirty tree before it reads the record.
The landing producers' "after the verify" clean check covers the dirty case for Publish and Finalize.
Separately, a ctx cancellation never surfaces as a non-zero exit with a nil error: `verifyrun.Run` checks `ctx.Err()` after any `cmd.Run` error and returns it, so that half of the directive is not a defect.

**Fix:** After a pass, re-read HEAD's tree and write the record only when it still equals the tree the run started on, so a moved HEAD costs a re-run rather than a false record.

### [LOW] DirtyPaths mangles quoted and arrow-bearing paths

**Location:** `internal/verifytree/verify.go:96` (`DirtyPaths`).

**Issue:** `git status --porcelain` (v1, text) C-quotes a path with special or non-ASCII bytes (`"caf\303\251.go"`, `"a\"b"`), and `strings.Trim(path, "\"")` strips only the outer quotes, leaving the escapes.
Any line containing ` -> ` is cut at it, so an untracked or modified file whose name contains ` -> ` is reported by its tail alone.
The paths reach Publish and Finalize's Stuck reasons, the gate findings and the fixer prompt verbatim, and the fixer is told to "commit or remove exactly those paths".
The non-empty check itself is unaffected.

**Fix:** Run `git status --porcelain -z` and parse NUL-separated records, taking a rename or copy entry's destination from its own record and skipping the source record that follows it.

### [LOW] Stale prose about the retired integration stage

**Location:** `internal/friction/friction.go:35`, `internal/friction/doc.go:36`, `contracts/stencils/friction/friction-directive-implementer.md:3`, `internal/loomshed/webster.go:89`, `internal/loomcli/wiring.go:520`, `internal/websterengine/report.go:71`.

**Issue:** The focus asked to look past the empty grep for prose that still describes the retired mechanisms.
The `RoleImplementer` doc, its `friction/doc.go` twin and the implementer friction stencil's banner name "the webster integration fork" as a consumer, where the consumer is now the verify-gate fixer fork (`RenderVerifyFixPrompt` renders `friction.RoleImplementer`).
`loomshed/webster.go` and `loomcli/wiring.go` say Merriam writes "the integration report" into the webster directory, which is now the verify-gate report the gate writes.
`validateReportFields`' doc names `ParseIntegrationReport`, which card 17 deleted.
The stencil registry rows, `claims_test.go`, the `envkit` nil-legal list and `gitrepoPinnedRunBoundMethods` are clean, and `state_test.go` pins that a `state.json` with `integrationFix` and a `-1` batch key still loads.

**Fix:** Reword each site to name the verify-gate fixer fork, the verify-gate report, and `ParseReport` alone.

### [NIT] Comment Line-Break Convention

**Location:** `internal/websterengine/verifygate.go:107`, `internal/verifytree/verify.go:31`, `internal/verifytree/verify.go:176`, `internal/webstercli/verify.go:3`, `internal/websterengine/cardgate.go:55`, `internal/landingshed/finalize.go:338`, `internal/landingshed/publish.go:257`, `internal/landingshed/verifygate_test.go:32`, `internal/mergeresolve/spec.go:22`, `internal/mergeresolve/mergeresolve.go:160`, `internal/mergeresolve/doc.go:19`, `internal/shuttleengine/wait.go:576`, `internal/loomcli/driverprompt.go:28`, `internal/websterengine/gitwrap.go:249`, `internal/websterengine/render.go:158`, `internal/websterengine/render.go:264`, `internal/websterengine/verifygatereport.go:60`, `internal/websterengine/verifyparse.go:59`, `internal/websterengine/verifygate_test.go:22`, `internal/websterengine/verifygate_test.go:24`, `internal/websterengine/verifyparse_test.go:165`, `internal/websterengine/runlevel_test.go:1584`, `internal/loomshed/seam_enforcement_test.go:50`, `internal/websterengine/doc.go:299`.

**Issue:** CONSTRAINTS.md requires one sentence per line and a break at an independent-clause boundary in every new or changed Go comment.
These new or rewritten comments join two independent clauses with a semicolon on one line, or put two sentences on one line inside a newly written fixed-column paragraph (`mergeresolve/doc.go`, `mergeresolve/spec.go`, `clearStaleReports`, the recover-batch paragraph of `websterengine/doc.go`).
No line-width finding is raised.

**Fix:** Break each at the semicolon or sentence boundary, rewrapping the newly written paragraphs one sentence per line.

### [NIT] Config comment understates a zero verify_gate_attempts

**Location:** `internal/websterengine/config.go:81`.

**Issue:** The changed comment says `VerifyGateAttempts` at 0 "leaves the verify gate no attempt".
`shuttleengine.GateEntry.Attempts` documents 0 as "the entry is off: it is skipped at every arrival", so a zero would silently drop the must-pass verify, which is why the knob is validated.

**Fix:** Say that 0 turns the must-pass verify gate off.

## Directive verdicts

- Verify directory placement: not a defect.
  In hub mode `<anchor>/.lyx` is the junction fabric excludes through `.git/info/exclude` with an anchored pattern (`excludePatternFor`), and in standalone mode `verifytree.Dir(stateDir)` sits outside the target repository, so the marker, log and record never reach `DirtyPaths`.
  The sites sharing one directory run at different times on one worktree (each gate at its own turn end, landing after both), and the fixer stencil forbids `lyx webster`, so concurrent `Verify` calls clobbering each other's marker or log need an operator running the verb during a gate.
- `NewVerifyGate` arrival order matches card 15 and the doc comment: non-done passes, dirty fails, the commit check is `Terminal` once a pre-fix head is recorded, no section passes with a warning, then verify, one rerun and a flaky pass.
  The first failure of any kind records HEAD.
  The closure's attempt counts evaluations; with the one entry, every evaluation but the last fails, so it tracks shuttle's consecutive-failure count.
  A non-done outcome after a failure ends the run as stuck or paused, never as done, so it bypasses nothing.
- Gate parity: `verify_parity_integration_test.go` drives `loomshed.NewVerifyGate` and `websterengine.NewVerifyGate`, the closures the recipe and `Run` wire, and the invariant's tripwire is `cmd/lyx/verifiedtree_test.go` as CONSTRAINTS.md says.
- Landing: Publish and Finalize check the tree before the merge-in, after it and after the verify, and every Stuck or error returns before `PushBranch` or the parent opener; `StatusSkipped` comes only from a record matching the tree and command.
- Merge stage: `git add -u` stages only tracked edits, the untracked check runs before any staging, and stale reports are cleared after the in-progress-merge abort and before attempt 1 (see F1 for the weft scope).
- Shuttle and launch: a gated run's `EventWaiting` batch returns no outcome; the deadline and liveness paths still finalize.
  The launch line carries `launchPointer` only, on both fresh and resume lines, and `RecoveryWaitBudget` outlasts the classify timeout measured from spawn.
- Card 12: `DeleteOnly` and `NamesGo` follow the card, and a card whose only Go targets were deleted renders no integration step.
- Card 17's Delete group: none of its deleted files or their basenames are referenced outside `_lyx`, and the build passes.

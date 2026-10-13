# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close six Webster faults that stop or mislead runs outside the run-exit audit:

1. A backgrounded `lyx gate test` shell is cut off when Master's run finishes, so a run can read `done` with a gate tier unfinished (#505, #509, #517).
2. Master fixes a verify-gate failure itself instead of spawning the one fixer fork.
3. A second recovery of a batch treats the first recovery's uncommitted files as foreign, and `recover-batch` exits 0 on a batch that ended terminal and not done (#522).
4. The recovery prompt does not say which gates it may run, nor what to do with a `tmux`-tagged step.
5. A `create-not-done` refusal does not show quarry's answer, so a missing symbol cannot be told from a resolver limit (#511).
6. A member declared once per mutually exclusive build-constraint set, or once per GOOS, resolves `ambiguous` and can never pass a Create, Edit or Delete check (#500, #507).

## Scope

In: the decisions below, and the docs they touch: the `doc.go` of each changed package, `contracts/specs/refusal-spec.md` (D5's new row, and any row quoting a text D7 or D8 changes), and `contracts/specs/loom-plan-spec.md` where it states the done-check and `glyph-ambiguous` rules.

Out:

- Quarry itself: no change to quarry's resolver and no quarry version bump.
- The orch and `CLAUDE.md`: their gate-test wording is unchanged.
- A Rename whose old side is a disjoint-ambiguous member: it stays refused, as a multipart old side is today.
- A card that deletes every declaration of a name and creates the same name again (`create-already-exists`, the second half of #507): with glyph Edit and Delete working for per-GOOS members, that plan is an Edit.
- Reviving a dead batch through `lyx loom goto` (#511's third point): not in the board entry.
- `lyx quarry resolve`'s own exit status on an `ambiguous` answer.
- The implementer's `deviations` list wording, which `arbiter-fixes` owns.

## Decisions

### D1. Webster's sessions run `lyx gate test` in the foreground

The implementer body (`contracts/stencils/webster/webster-body-implementer.md`), which forks and recovery strands both read, says each card gate step, each card `verify:` line and each batch gate step runs in the foreground with the Bash tool timeout set to 3600000, never in the background, replacing its `run_in_background`, waited-for wording.
The steer the claude engine announces with the raw-go deny (`steerRawGoDeny`) drops "as a background Bash call" and names no mode, so it no longer contradicts the body and leaves other sessions' practice to their own instructions.

Rationale: the cut-off shells were `lyx gate test --tags integration` runs a session backgrounded because the steer and the body told it to.
A fork that backgrounds a gate and returns leaves the shell in Master's session, where nothing reads its result.
A foreground call keeps the result in the turn that needs it.
The Bash timeout then bounds the slot wait, at most the hub's `cli_wait_sec` before exit 75, plus the test run.

### D2. A `lyx gate test` shell holds Master's turn end, and an unheld one is recorded unread

Master's spawn declares `lyx gate test` as a second awaited shell prefix beside `lyx webster recover-batch` (`masterAwaitedShellPrefix`).
Shuttle's prefix match and turn-end rules stay as they are: an awaited shell holds even a gated run's turn end with every output file present.
Webster's `webster-background-shell` note and warning state, for each ended shell, that the run never read its result.

Bound: the hold lasts until the shell ends, bounded by Master's run timeout and the liveness check, as the recovery prefix's hold is today, and it applies whatever outcome Master wrote, so a stuck or paused Master also waits for the shell.
A shell whose label does not start with `lyx gate test` is not held: one wrapped in `cd … &&`, or one the provider reports with no command, whose label falls back to its description.
Such a shell ends as today, and the note and warning say its result was not read.

Rationale: D1 removes the cause; this is the backstop for a session that backgrounds a gate anyway, using the mechanism shuttle already has for the recovery verb.

### D3. The verify gate's re-prompt sends Master to the fixer-fork procedure

`shuttleengine.GateEntry` gains an optional re-prompt line builder, taking the findings path and returning one line; an entry without one keeps shuttle's generic `Gate findings recorded at … — read it, fix every finding, and end your turn.`
Webster's `verify` entry (`NewVerifyGate`) sets a line that keeps the `Gate findings recorded at <path>` opening and replaces "fix every finding" with an order not to fix them itself and to follow its "A gate failure: spawn one fixer fork" section.
The line is validated as one line, like the generic one.

Rationale: the generic line orders Master to fix the findings itself, which contradicts its stencil at the moment Master reads the failure.

### D4. Recovery sessions are part of the run's write evidence

`recover-batch` records the recovery strand's session id, which the engine mints at launch, in a run-level list in `state.json` at every spawn.
`loadRunWrites` reads those sessions too and keeps their writes in a third group beside Master's and the forks', so the contract-file evidence, which reads Master's writes, and the run-exit audit, which reads `recordedSessions`, are unchanged.
The recovery prompt's "Written by this run" class then holds every path an earlier recovery of this run wrote.
The list survives a re-begin, a rebaseline and every reset but `reset --to start`, which archives the whole record, as `run --fresh` does.
A recorded recovery session whose transcript is missing (a spawn that never came up, a resume on another machine, a pruned transcript) is no evidence, never an error, so its paths fall under "Not written by this run".

Bound: attribution widens only to sessions the run itself spawned and recorded; a path no recorded session wrote stays under "Not written by this run".
A state written before the field existed reads as an empty list, which is today's behavior.

Rationale: write evidence today covers Master and its forks only, so an earlier recovery's work reads as foreign and the next recovery blocks on it.
Attribution by the run matches what the class already means for fork leftovers.

### D5. `recover-batch` exits non-zero unless the batch ends done

A terminal digest whose status is not `done` (stuck or dead, any `dead_reason`) returns an error envelope carrying the same digest fields as today, `recovery_retry` and `recoveries` included.
The error names the batch, the status and the way forward: run `recover-batch` once more when `recovery_retry` is true, otherwise the run is stuck, with `lyx webster reset --to batch-start` or `--to start` as the operator's route.
It gets a row in `refusal-spec.md`'s webster section.
State persistence and the fabric sync happen before the output as today, and a sync failure keeps its existing error.
A `done` digest and a `--wait` running snapshot still exit 0.
Master's failure ladder in `webster-template-master.md` says a stuck or dead digest arrives as this error with its fields, and the rungs read the same fields; every other caller of `recover-batch` that reads its exit status or `ok` is checked against the change.

Rationale: exit 0 on a batch that ended terminal and not done reads as success to any caller that checks the exit status.

### D6. The recovery prompt names its gates and the `tmux` rule

`contracts/stencils/webster/webster-prefix-recovery.md` states, beside its ban on `lyx webster`:
the card gates and the batch gate it is given, `lyx gate test` and `lyx loom lint-comments`, are always allowed and are neither `lyx webster` nor mutating `lyx fabric` commands;
a card step tagged `tmux` is not run by the recovery and is left to the run's verify gates.

Bound: a recovery leaves a `tmux`-tagged step unchecked; the plan-level verify and the loom's later verify gates still run whatever tiers they cover, and the recovery's report does not claim that step passed.

Rationale: the recovery could not tell whether its gates fell under the fabric ban, and could only compile-check `tmux` steps.

### D7. A disjoint-ambiguous answer counts as one member

`internal/planglyph` gets one predicate that reads an `ambiguous` answer as a single member when no build context selects two of its candidates.
A build context is one GOOS, one GOARCH, and any assignment of the other tags the candidates' `//go:build` lines name.
GOOS and GOARCH names in a constraint are bound to the context, not free tags, and `unix` follows from the GOOS.
A file name's `_GOOS` and `_GOARCH` suffixes constrain it the same way.
A `_test.go` file overlaps every non-test file of its package, so a test-file and a non-test candidate are never disjoint.
A candidate file that cannot be read, or whose constraint does not parse, makes the answer overlapping, and the finding names the file and the reason.

Every site that accepts `found` or `multipart` today accepts such an answer the same way, with its candidates as the symbols:
the plan-time resolve pass (`glyph-ambiguous`), the Create collision check (`create-already-exists`), begin-batch's re-resolution and post-batch drift detection, done-checks for Create, Delete and the Rename new side, delete ordering, handle binding and the scope guard's target union.
An `ambiguous` answer whose candidates overlap stays blocking, and its finding lists each candidate's file with its constraint.
At Plan time this flags an existing member that can never pass; a Create target does not exist yet, so its overlap shows first at record-batch.

Bound: the widening admits only candidates of one glyph whose constraints are pairwise disjoint, which Go's compiler requires of such a member; two declarations one build compiles together still block.
Non-Go plans are unaffected.

Rationale: quarry ignores build constraints, so the idiomatic per-tag or per-GOOS member answers `ambiguous` and no plan or batch can name it; one predicate fixes every consumer.

### D8. `create-not-done` shows quarry's answer

The `create-not-done` detail names quarry's status for the target, its unit status on `not_found`, and on `ambiguous` each candidate's file and constraint, and says the lookup was `quarry resolve` over the worktree with test files and unexported members included.

Rationale: "still does not resolve" hides whether the symbol is missing, misplaced or ambiguous.

## Constraints

- `PATTERN-shuttle-provider-seam`: D1's steer change stays inside `internal/shuttleengine/claudeengine`; D3's re-prompt builder is provider-neutral.
- `PATTERN-stencil-ownership`: D3's line is a one-line Go message beside shuttle's own generic line, the same class as a refusal text, not a prompt file; the procedure it points at stays in Master's stencil.
- `PATTERN-refusal-way-forward`: D5's error is a new webster refusal, so it names its way forward, lands its `refusal-spec.md` row in the same commit and a test reaches it.
- `PATTERN-no-denied-recovery`: no new text names a denied command; foreground `lyx gate test` is the allowed route.
- `PATTERN-glyph-conversion-chokepoint`: D7 reads the `file` quarry returns and parses no glyph.
- `PATTERN-documentation-lifecycle`: the changed behavior lands in the `doc.go` of each changed package.

## Auto-mode assumptions

- D1 scopes the foreground rule to Webster's stencils; the shared steer only stops naming a mode.
- D2 matches by label prefix, as shuttle does today; no shell-segment parsing.
- D4's attribution is by run, not narrowed to the batch being recovered, matching how fork leftovers are classed.
- D6's `tmux` rule sits in the recovery prefix only, as the board asks.
- The board's "flag at Plan time" half is D7's narrowed `glyph-ambiguous` check.
- #511's `cmd/testtiming#rusageStats` failure stays unexplained: the landed code has one declaration, which quarry answers `found`. D8 makes the next such failure show its cause.

## Open risks

- A gate whose slot wait plus run exceeds the Bash tool's 60-minute ceiling is killed in the foreground; `cli_wait_sec` defaults to 300 seconds.
- D6 leaves a `tmux`-tagged card step unchecked until a verify gate that covers the `tmux` tier runs.
- D7 enumerates build contexts; a constraint naming many custom tags grows that enumeration exponentially, though real constraints name a handful.
- `arbiter-fixes` makes `ScopeGuard`'s result the deviation list readers see; D7 adds every candidate file of a disjoint-ambiguous target to the guard's union, so a per-GOOS sibling stops reading as a deviation, and whichever task lands second keeps both rules.

## Acceptance criteria

- A gated, fresh Master run with every output file present and a single outstanding shell labelled `lyx gate test --tags integration ./...` does not finish at that turn end; it finishes once the shell has ended and a later turn end arrives at the gate.
- The same holds when Master's outcome file says `stuck`.
- A shell with any other label finishes as today, and the background-shell note and warning say its result was not read.
- The implementer body directs every gate and `verify:` step to the foreground with timeout 3600000, and the announced steer names no background mode.
- A failed `verify` gate entry sends Master a line naming the findings file and the fixer-fork section, without "fix every finding"; a gate entry with no builder still sends the generic line.
- A second recovery of a batch lists the files the first recovery wrote, uncommitted, under "Written by this run", while a file no recorded session wrote stays under "Not written by this run".
- A recorded recovery session with no transcript yields no evidence and no error, for the recovery prompt and for a reset.
- Recovery sessions add no parent-write finding to the run-exit audit and no evidence to the contract-file check.
- A state file without the recovery-session field loads and classes paths as before.
- `recover-batch` exits non-zero with the digest fields and a way forward on a terminal stuck digest and on a terminal dead digest, and exits 0 on done and on a running snapshot.
- The rendered recovery prompt names `lyx gate test` and `lyx loom lint-comments` as allowed and states the `tmux`-step rule.
- A member declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`) passes a Create done-check, and an Edit or Delete names it at Plan time and at begin-batch without `glyph-ambiguous`.
- A member with `_linux.go` and `_windows.go` bodies, one with `//go:build linux` and `//go:build !linux` bodies, and one with `//go:build unix` and `//go:build windows` bodies pass the same checks.
- Two candidates one build compiles together, a member in `x.go` and `x_test.go` of one package among them, still produce `glyph-ambiguous` at Plan time and `create-not-done` at record-batch, each listing the candidates' files and constraints.
- A candidate whose constraint does not parse or whose file is unreadable makes the answer blocking, with the file and reason named.
- A disjoint-ambiguous Delete target still present after its card fails `delete-not-done`.
- A `create-not-done` finding states quarry's status, the unit status on `not_found`, and the candidates on `ambiguous`.

## Notes for the plan writer

- The record faces base `800a0e892`; read it with `git show 800a0e892:<path>`.
- The gate re-prompt is `gateRepromptText` in `internal/shuttleengine/gate.go`; the verify entry is built by `NewVerifyGate` in `internal/websterengine/verifygate.go`.
- Write evidence is `loadRunWrites` and `recordedSessions` in `internal/websterengine/runwrites.go`; the prompt classing is `uncommittedPathsBlock` in `recoverbatch.go`; the claude engine's audit errors on a missing parent transcript.
- The terminal output of `recover-batch` is in `internal/webstercli/recoverbatch.go`.
- The `found`/`multipart` acceptance sites are in `internal/planglyph`: `resolve.go`, `create.go`, `donecheck.go`, `deleteorder.go`, `handle.go`, and the scope guard.
- The shuttle replay corpus case `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running` is a real Master turn end with a fork's leftover shell.
- New top-level tests that push a package past its row in `cmd/lyx/testdata/test-budget.yaml` list that file under `Edit:`.

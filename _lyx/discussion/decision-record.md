## Goal

Close six Webster faults that stop or mislead runs outside the run-exit audit, as the board entry `twb-ftb-1` lists them.
The premises below were read at base `800a0e892`.

1. A `lyx gate` shell Master backgrounds can be cut off by a done turn end (#505, #509, #517).
2. Master fixes a verify-gate failure itself instead of spawning the fixer fork.
3. A second recovery of a batch treats the first recovery's uncommitted files as foreign (#522), and `recover-batch` exits 0 over a batch that ended dead with no recovery left.
4. The recovery prompt does not name the gates it may run.
5. `create-not-done` reports a resolver limit as a missing symbol (#511).
6. A Go member declared once per disjoint build-constraint set resolves `ambiguous`, so its Create, Edit or Delete can never pass (#500, #507).

## Scope

In:

- `internal/websterengine`: Master's awaited shell prefixes, the verify-gate re-prompt text, the background-shell warning wording, the recovery write evidence.
- `internal/webstercli`: `recover-batch`'s exit status on a terminal dead digest with no recovery left.
- `internal/shuttleengine`: an optional per-entry re-prompt text on `GateEntry`.
- `internal/planglyph`: one build-variant predicate over ambiguous answers, used by the status policy, the delete-order reader and `DoneChecks`; the done-check detail text for `create-not-done` and, as the same Create direction, a Rename's new side.
- Stencils: `contracts/stencils/webster/webster-prefix-recovery.md`, `contracts/stencils/webster/webster-template-master.md`.
- `contracts/specs/refusal-spec.md`: the webster rows D6 and D10 add or change.
- Docs in the same commits: `internal/websterengine/doc.go`, `internal/shuttleengine/doc.go`, `internal/planglyph/doc.go`.

Out:

- Changing quarry (`github.com/Knatte18/quarry`); it stays at its pinned version and keeps ignoring build constraints.
- A Rename whose old side is a build-variant set: `renameDeclSource` stays found-only.
- #507's second half, a `Create` handle reusing a name the same card deletes.
- #511's third point, reviving a dead batch through `lyx loom goto`.
- Shells a fork backgrounds: a running fork already holds Master's turn end.
- The reset's own-path set: it keeps reading Master and fork writes only.
- Languages other than Go: the build-variant predicate applies only when the plan's glyph language is Go.

## Decisions

### D1. Master's wait holds on a `lyx gate` shell

Master's spawn spec awaits a second shell prefix, `lyx gate `, beside `lyx webster recover-batch`.
Rationale: at base no shell expires, but `countsAtOnce` still counts a gated, fresh Master turn end as done when its output files exist and every outstanding task is an unawaited shell.
A backgrounded `lyx gate test --tags integration …` was exactly such a shell, so the run finished done and strand removal killed it.
An awaited shell holds the turn end like a fork, and its completion notification gives Master a turn that re-arrives at the gate.
Bound: the hold covers only a shell whose command label starts with `lyx gate `; it ends when the shell exits, at Master's `master_timeout_min` deadline, or at the liveness check, as an awaited `recover-batch` shell already does.
A gate command not leading its label, such as one behind `cd … &&`, stays unawaited and keeps today's behaviour; D2's stencil line is its only guard.

### D2. Master's stencil names `lyx gate` as a command to lead, never wrap

`webster-template-master.md` says that a `lyx gate` command Master backgrounds is written with `lyx gate` first in the command, so the wait can see it, and that Master never runs a gate to check a fixer's work, since the verify gate re-runs the plan's verify on the next arrival.
Rationale: D1 matches by prefix, and the stencil is what keeps the label matching; the second sentence removes the reason Master ran integration gates itself in `driver-context`.

### D3. Ended-shell warning says the result was not read

`expiredShellWarning` adds that the shell's result was never read by the run.
Rationale: the board asks that an outcome never read as "tests passed"; the warning and friction note already carry the shell, and this states the consequence in the same line, on every outcome that ends over a shell.

### D4. The verify gate's re-prompt sends Master to the fixer fork

`shuttleengine.GateEntry` gains an optional re-prompt text, a function of the findings path; a nil one keeps `gateRepromptText`'s default.
Webster's `verify` entry sets it to one line opening `Gate findings recorded at <path>`, that says Master does not fix them itself and spawns one fixer fork with the rendered verify-fix prompt, ends its turn, and rewrites its outcome and summary on the fork's notification.
Rationale: at base the verify gate re-prompts with shuttle's generic "read it, fix every finding, and end your turn.", which tells Master the opposite of its stencil at the moment it reads the failure.
Bound: the custom text replaces only the sent line; it stays one line under `validateSendText`, keeps the `Gate findings recorded at` opening the stencil keys on, and changes no attempt count, verdict or other gate entry.

### D5. Recovery write evidence includes every recovery strand of the run

Each recovery spawn persists its strand's session id on the batch record once shuttle reports it, and both a respawn and a re-begin carry the earlier ids onto the fresh record, as `Recoveries` is carried.
The write evidence audits those sessions by session id, never through `ShuttleRunDir`, and returns their writes as a group beside Master and forks.
Only `uncommittedPathsBlock` reads that group; the contract-file evidence and the reset's own-path set keep reading Master and fork writes.
Rationale: an earlier recovery's unfinished work then lands under "Written by this run", and the strand keeps it as its starting point.
Bound: a recovery whose session id was never recorded, or whose transcript cannot be read, contributes nothing, so its paths stay under "Not written by this run", and the prompt block adds one line naming the batch's recovery whose writes could not be read; the render never fails on it.

### D6. `recover-batch` exits non-zero when a dead batch has no recovery left

A terminal `dead` digest with `recovery_retry` false is returned as an error envelope carrying the same fields as today's success envelope (the digest fields, `recovery_retry`, `recoveries`, `warnings`), after the same persist and fabric sync.
Its message and its new row in `contracts/specs/refusal-spec.md` reuse the existing ladder's wording for that case, "any other terminal dead: end the run stuck naming the batch", so the two cannot drift.
A `done` digest, a `stuck` digest, a `dead` digest with `recovery_retry` true and a `--wait` that elapses keep exit 0; the `batch_failed` path keeps its own envelope.
Rationale: at base the `ErrBatchFailed` path already exits non-zero, and a stuck or retryable dead digest's next step is another `recover-batch`; the one terminal end that returned exit 0 is the dead batch the ladder ends the run on.
Master's stencil reads the same fields from the error envelope.

### D7. The recovery prompt names its allowed gates

`webster-prefix-recovery.md` states that every step its card gates and batch gate render, every line of a card's `verify:`, and `lyx quarry resolve` are always allowed and are none of the `lyx fabric` verbs the sandbox rule bans.
A card's `verify:` line tagged `tmux` is not run by the strand, which says so in its report; the plan's verify compiles that tier and Publish runs it.
Rationale: the strand in `quick-recipe` was unsure whether these fell under the fabric ban.
The text names no step at all for the `tmux` line, so it neither contradicts the implementer body's "a `tmux`-tier run goes through `lyx gate test`, never a raw `go` command" nor names a denied form (PATTERN-no-denied-recovery).

### D8. One build-variant predicate reads an ambiguous answer

`internal/planglyph` gains one predicate: an `ambiguous` answer is a build-variant set when every candidate sits in one package directory and no build configuration selects two of the candidates' files.
A file's constraint is decided by `go/build`'s own matching rules, `//go:build` and legacy `// +build` lines and `_GOOS`/`_GOARCH` file-name suffixes, with its implied GOOS values (`android` builds `linux` files, `ios` builds `darwin`, `illumos` builds `solaris`) and `unix`.
Disjointness is checked over every known GOOS/GOARCH pair and every assignment of the other tags the candidates' constraints name.
A candidate file the predicate cannot read or parse makes the set not a variant set.
The predicate reads candidate files under the worktree root its caller passes, and resolves no working directory (PATTERN-told-geometry, PATTERN-cwd-resolution).
Rationale: quarry ignores build constraints and answers `ambiguous` for `tmuxAllowed` (`allow_tmux.go`, `allow_none.go`) and `StartTime` (`proc_linux.go`, `proc_windows.go`); one Go-only predicate in lyx fixes every reader without a quarry change.
Bound: the predicate admits only declarations no single build can see together, so two declarations a build compiles together stay ambiguous; it never applies to a non-Go plan.

### D9. Every present-reading of an ambiguous answer accepts a build-variant set

- `statusFindings` passes a build-variant set like `found`, at Plan and in record-batch's drift pass, which also calls it, so an Edit, Uses or Delete target that is one draws no `glyph-ambiguous`.
- `answerSymbols` returns every candidate's symbol for a build-variant set, so the delete-order guard sees a per-GOOS symbol a batch deletes while a later card still references it.
- `DoneChecks` reads a build-variant set as resolved for `create-not-done` and for a Rename's new side.

`delete-not-done` and `create-already-exists` keep treating any ambiguous answer as present.
Rationale: these are the sites the board's items 5 and 6 hit, and the board's Plan-time flag is met by the same predicate: a glyph Plan can resolve is judged there, and a non-variant ambiguous glyph is still refused at Plan.
A Create handle names a member that does not exist at Plan time, so no candidates exist to flag; record-batch is the first point it can be read, and it now accepts the variant set.

### D10. The done-check refusal says how the target was looked up

`create-not-done` and the Rename new side's `rename-not-done` detail name the key resolved with `lyx quarry resolve`, the status quarry answered, and:

- for `not_found`, whether the unit resolved, and that quarry reads every Go file of the package, test files and unexported names included;
- for `ambiguous`, each candidate's file and why the set is not a build-variant set (two candidates a build compiles together, or a file that could not be read).

For an `ambiguous` answer, `recover-batch NN` alone repeats the failure, so the reason names the plan edit as the way forward: change the card so the member has one declaration or a build-variant set, `lyx webster rebaseline --card NN`, then `lyx webster recover-batch NN`; the `card not done` row in `contracts/specs/refusal-spec.md` gains that branch.
Rationale: probing at quarry `v0.2.1`, the base pin, unexported members and test-file members resolve `found`.
The inference is that #511's targets failed as per-GOOS or per-tag declarations under a detail that hid the `ambiguous` status; the run's tree is gone, so the detail is what settles such a case next time.

## Constraints

- PATTERN-shuttle-provider-seam: D4's hook is generic shuttle code; the fork wording lives in `websterengine`'s entry text, never in `shuttleengine`'s default line.
- PATTERN-refusal-way-forward: D6's new row and D10's changed row land with their code, each with a test reaching it.
- PATTERN-no-denied-recovery: D6's, D7's and D10's texts name no denied form.
- Docs land in the same commit as their code: `internal/websterengine/doc.go` (its "Background shells in Master's wait", "The verify-gate fixer fork" and recovery-prompt paragraphs), `internal/shuttleengine/doc.go` (gate re-prompt), `internal/planglyph/doc.go` (the build-variant rule and the check IDs it touches).

## Auto-mode assumptions

- Item 1's "a running gate shell holds the turn end" is met by D1.
- Item 2's two options are met by stating the rule in the re-prompt Master reads at the failure (D4).
- Item 3's "attribution by batch, not by shuttle run id" is met run-wide (D5), matching the run-wide attribution Master and fork writes already have.
- Item 3's "exited 0 while the batch ended failed and terminal" is read as the dead digest with no recovery left (D6).
- Item 5's "find whether" is answered in D10's rationale.
- Item 6's Plan-time flag is met as D9 states.

## Open risks

- An awaited `lyx gate` shell that hangs holds Master until `master_timeout_min`; the gate's own slot and timeout normally end it first.
- D6 changes `recover-batch`'s exit contract; `weaver`, planned to reuse `recover-batch`, must read a dead-with-no-retry end from the error envelope.

## Acceptance criteria

- Master's shuttle spec awaits shells whose label starts with `lyx gate ` and `lyx webster recover-batch`.
- A gated, freshly started run whose waiting turn end has every output file present and an outstanding shell labelled `lyx gate test --tags integration ./x | tail` is not counted done; after the shell's completion, the next turn end arrives at the gate.
- The same turn end with only an unawaited shell outstanding still counts done, and the run's warning for that shell says its result was not read.
- A failed `verify` gate entry sends Master one line opening `Gate findings recorded at`, naming the fixer fork and the verify-fix prompt, with no instruction to fix the findings itself; another gate entry with no custom text still sends the default line.
- A second recovery of a batch, after a first recovery strand wrote uncommitted files and was stopped, renders those files under "Written by this run"; the same holds after a re-begin.
- A recovery whose session id was never recorded, or whose transcript is unreadable, leaves its paths under "Not written by this run", adds the naming line, and the recovery still spawns.
- The contract-file evidence and the reset's own-path set are unchanged by recovery-strand writes.
- `recover-batch` exits non-zero with the digest fields, `recovery_retry` and `recoveries` on a terminal dead digest with `recovery_retry` false, and exits 0 on done, stuck, a retryable dead digest and an elapsed `--wait`.
- The rendered recovery prompt names the card gates, the batch gate, a card's `verify:` lines and `lyx quarry resolve` as allowed and outside the fabric ban, and says a `tmux`-tagged `verify:` line is left to the plan's verify and Publish.
- A Create of a member declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`) passes `DoneChecks`.
- A Create of a member declared in `x_linux.go` and `x_windows.go` passes `DoneChecks`, and an Edit of it passes the plan-time status policy.
- A member declared in `x_linux.go` and `x_android.go` stays ambiguous.
- A member declared in two unconstrained files of one package stays `create-not-done` and `glyph-ambiguous`, with a detail naming both files, that a build compiles them together, and the plan-edit way forward.
- A Delete of a build-variant set with one variant left still draws `delete-not-done`, and a later card referencing a deleted build-variant set is caught by the delete-order guard.
- A `create-not-done` over a `not_found` answer names the resolved key, the status, whether the unit resolved, and that test files and unexported names are read.
- A non-Go plan's ambiguous answer is judged as before.

## Notes for the plan writer

- Master's spawn spec and `masterAwaitedShellPrefix` are in `internal/websterengine/runlevel.go`; the at-once count is `countsAtOnce` in `internal/shuttleengine/wait.go`.
- The default re-prompt is `gateRepromptText` in `internal/shuttleengine/gate.go`; the verify entry is built where `NewVerifyGate` is appended in `runlevel.go`.
- The ended-shell wording is `expiredShellWarning` in `internal/websterengine/triage.go`.
- Recovery evidence: `loadRunWrites` and `recordedSessions` in `internal/websterengine/runwrites.go`, `uncommittedPathsBlock` in `recoverbatch.go`, the batch record in `state.go`.
- `recover-batch`'s terminal envelope is near the end of `internal/webstercli/recoverbatch.go`'s `RunE`.
- Planglyph sites: `statusFindings` (`resolve.go`, also called from `drift.go`), `answerSymbols` (`deleteorder.go`), `doneCheckVerdicts` (`donecheck.go`).
- A card whose new top-level tests push a package past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.

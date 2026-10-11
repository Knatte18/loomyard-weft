# Decision record — twb-tw-om-1: Webster stability outside the audit

## Goal

Close the six Webster faults the board entry names, each of which stopped a run or let it report a misleading outcome:
a background gate shell cut off at a turn end (#505, #509, #517), Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign and `recover-batch` exiting 0 on a failed batch (#522), a recovery prompt that does not name its allowed gates, Create checks whose refusal cannot tell a missing symbol from a resolver limit and a dead batch that blocks its own way forward (#511), and Create and Edit checks that cannot pass for a member declared once per build-constraint set or per GOOS (#500, #507).

## Scope

In:

- `internal/shuttleengine`: the gated turn-end rule for outstanding background work.
- `internal/shuttleengine` and `internal/websterengine`: the verify gate's re-prompt and findings text.
- `internal/websterengine`: recording recovery sessions, counting their writes as the run's own, and the ended-shell wording.
- `internal/webstercli`: the `recover-batch` exit status.
- `contracts/stencils/webster/`: the implementer body's allowed-commands and tagged-test lines, and Master's dead-batch rungs.
- `internal/planglyph`: done-check finding detail and the partitioned-member classifier, with every status-policy site that reads an `ambiguous` answer.
- The module docs (`doc.go`) of each package above, and the `contracts/specs/refusal-spec.md` and `contracts/specs/loom-plan-spec.md` rows the changed findings and refusals touch.

The board's "plan validator flags such a glyph at Plan time" is met by `glyph-ambiguous` for Edit and Delete targets (`D7`); a Create target's future ambiguity does not exist at Plan time, so `D6`'s detail covers it at record-batch.

Out:

- quarry and `lyx quarry resolve`: answers stay raw.
- A lyx verb that spawns the verify fixer.
- A waiver for a Delete-then-Create of one name.
- Reviving a dead batch through `lyx loom goto` (`D8` names the reset route instead).
- The fabric audit, which `audit-reader-globs` and the audit work own, and how verify itself runs.

## Decisions

### D1 — A gated run's turn end holds on any outstanding background task

Decision: `countsAtOnce`'s exception goes.
A gated run's waiting turn end with every output file present never counts as an arrival while any background task, a shell or a fork, is outstanding; the run waits for the task's completion, as it already does for a fork.
`Spec.AwaitedShellPrefixes`, Webster's `masterAwaitedShellPrefix` and the `startedFresh`/`gatedArrival` fields are removed, since every shell now holds a gated run and `countsAtOnce` was their only reader.
Ungated runs keep today's rule: every output file present counts at that turn end, whatever is outstanding.
A held turn end (output file missing, nothing outstanding) is outside this decision.
Bound: the hold is bounded by the run's own deadline (`master_timeout_min` for Master) and the liveness check, never by `background_shell_wait_min`, which stays a logging and wait-marker threshold.
A gated run held on a shell that never ends, or whose completion shuttle never sees, reaches its deadline with every output file present, and `classifyDeadlineExpiry` ends it `OutcomeDone` per `PATTERN-completion-signal`; finalize runs the gate once, and the shell lands in `Result.EndedShells`.
Webster's ended-shell warning, friction note and summary section change their wording to say each listed shell's result was never read, so a run ended over a shell never reads as "tests passed".
Rationale: every agent is told to run `lyx gate test` backgrounded, so any gated producer, not only Master, can hit the fault.

### D2 — The verify gate's re-prompt and findings text open with the way forward

Decision: `GateEntry` gains an optional re-prompt renderer taking the findings path and returning the one-line message.
An entry without one, or whose renderer returns empty text, sends the shared default, `Gate findings recorded at <path> — read it, fix every finding, and end your turn.`
Webster's `verify` entry supplies one line that keeps the `Gate findings recorded at <path>` opening, which Master's stencil and the send guard key on, and then says: follow your gate-failure section, spawn the one fixer fork, and never fix, edit or commit anything yourself.
The findings text the gate returns to Master (`renderVerifyGateFindings`) opens with the same line; the YAML report the fixer fork reads (`verify-gate.yaml`) is unchanged.
This restates Master's stencil rule at the point of failure, a departure from `PATTERN-producer-pointer-rule` the board asks for; the line points at the stencil section and adds no rule of its own.
Both strings are Go-authored runtime messages like the default re-prompt and the refusals, not producer prompts, so `PATTERN-stencil-ownership` does not bind them.
Rationale: the default re-prompt tells Master to "fix every finding", contradicting its stencil at the exact point Master reads the failure.

### D3 — Every recovery session is recorded on its batch, and its writes are the run's own

Decision: `BatchState` gains a list of recovery session ids.
`recoverSpawn` appends the spawned strand's session id once its start succeeds, before the wait; a start that fails records none, a pure attach adds none, and the list holds no duplicates.
A recovery respawn and a re-begin carry the list onto the fresh record, as `Recoveries` and `AuditWarnings` are carried.
`RunWrites` gains a third group, recovery writes: each recorded recovery session's own writes and its forks' writes, audited through the same engine call as a Master session.
Recovery writes count as the run's own in all three readers:

- the recovery prompt's uncommitted-path classing (`uncommittedPathsBlock`) lists them under "Written by this run";
- a reset's own tracked paths (`ownTrackedPaths`) include them;
- contract-file evidence treats a recovery write like a fork write: a contract file whose last write came from a recovery stays uncleared unless a later successful Master write exists, and the finding's message names the last writer as a fork or a recovery session.

Recovery sessions never join Master's run-exit parent-write audit: their own writes are not Master's.
A recovery session whose transcript is missing or unreadable is skipped with a logged warning: its paths stay foreign in the prompt and are not discarded by a reset, and contract-file evidence reads every contract file as uncleared while any recovery session was skipped.
Bound: a reset may now discard a tracked path a recovery wrote; it is bounded to a successful write recorded in the transcript of a session listed on one of this run's batch records, and a path no such session wrote stays foreign.
A record written before the field exists reads as no recovery sessions, today's behaviour.
Rationale: a recovery strand is a separate session no state field records, so `recordedSessions` never audits it and its uncommitted work reads as foreign to the next recovery; attributing by batch keeps the first attempt's work.

### D4 — `recover-batch` exits non-zero on every terminal batch that is not done

Decision: a terminal digest whose status is `failed`, `stuck` or `dead` makes `lyx webster recover-batch` exit non-zero with `ok: false`, carrying the same digest fields (`status`, `recoveries`, `recovery_retry` and the rest) the success envelope carries, and no `batch_failed` field.
The terminal persist and its fabric commit run first, exactly as today, so the exit status changes nothing on disk.
The error message names the failure ladder's way forward for that status.
A `--wait` running snapshot and a `done` digest still exit 0.
Master's ladder keys on the digest's `status` and `recovery_retry`; the stencil's rungs for those statuses are reworded only where they say "completes with" a status the call now reports as a refusal, and the `batch_failed` rung keeps firing only on that field.
Rationale: one rule, exit status agrees with the batch outcome, and no reader loses a field.

### D5 — The implementer body names its allowed commands and the tagged-test rule

Decision: `webster-body-implementer.md`, shared by forks and recovery strands, states:

- the card gate steps, the batch gate steps, `lyx gate test`, `lyx loom lint-comments` and `lyx quarry resolve` are always allowed;
- the fabric ban covers mutating `lyx fabric` verbs only, and the recovery prefix's `lyx webster` ban is unchanged;
- a `tmux`- or `llm`-tagged test run, whether a gate step, a card's `verify:` line or the agent's own, is compiled with `go vet -tags <tag> <package>...`, packages listed one by one with no `...` pattern, and never run; the plan's verify and Publish's `publish_verify` own those tiers.

Bound: only the `tmux` and `llm` tags are compiled instead of run; every untagged and `integration` step runs as listed.
Rationale: the recovery could not tell whether the gate verbs fell under the fabric ban, and the plan spec already leaves the `tmux` tier to Publish and never runs the `llm` tier; `steerRawGoDeny` refuses a tagged `go test` and any `...` pattern, but not a package-scoped `go vet`.

### D6 — Done-check findings say how the target was looked up

Decision: `create-not-done`, `delete-not-done` and `rename-not-done` name the resolve key looked up, quarry's status, and for `ambiguous` each candidate's file and its build constraint (file-name suffix or `//go:build` line).
Rationale: quarry resolves unexported members and members in `_test.go` files, so #511's refusals were most likely the plan's per-build-set declarations answering `ambiguous`; the old text read the same for that and for a missing symbol, and the new detail tells them apart.

### D7 — A partitioned member is one member

Decision: one classifier in `planglyph` decides whether an `ambiguous` answer's candidates are partitioned: every candidate file is selected by at least one build environment, and no single environment selects two of them.
It evaluates each candidate file with `go/build.Context.MatchFile` over every environment in a set of:

- every GOOS/GOARCH pair in a list pinned in `planglyph` (the toolchain's `go tool dist list`);
- cgo on and off;
- every true/false assignment of the free tags the candidates' constraints mention.

`Compiler` is set to a value that matches neither `gc` nor `gccgo`, and `ReleaseTags` and `ToolTags` are emptied; `gc`, `gccgo`, `go1.N` and `goexperiment.*` are free tags passed through `BuildTags` per assignment, so none is held true by the toolchain lyx was built with.
`unix` follows from GOOS through `go/build` itself.
Each candidate file is read once and served to `MatchFile` from memory through `Context.OpenFile`.
More than four free tags, an unreadable file, an unparsable constraint, or a candidate no environment selects leaves the answer `ambiguous`.
Two candidates one environment selects together, such as a package and its `x_test` package declaring the same name, are never partitioned.

A partitioned answer is accepted like `multipart` at every status-policy site: an Edit or Delete target, `Uses:`, a Rename's new side, the Create and Rename-new done-checks, delete-order's declaration list (each candidate's declaration) and the plan gate's member reads.
The plan gate and the record-batch done-check call the same classifier, per `PATTERN-gate-self-check-parity`.
`createFindings` still reads any `ambiguous` Create target, partitioned or not, as `create-already-exists`, and the Delete done-check still blocks while a partitioned member exists.
A Rename's old side stays found-only, since the new declaration is derived from one signature; its refusal names the way forward: a Delete of the old member and one Create of the new handle on a card that edits each declaring file.
A genuinely ambiguous Edit or Delete target stays blocked at Plan time by `glyph-ambiguous`, now listing each candidate's file and constraint.
Bound: acceptance applies only to candidates the classifier proves disjoint and each selectable over the whole environment set; every other `ambiguous` answer keeps today's blocking finding.
Rationale: `MatchFile` is the toolchain's own reading of file names and constraints, so the classifier cannot drift from what `go build` selects, and enumerating free tags keeps one build of lyx from deciding for another.

### D8 — A dead batch names the reset route

Decision: `lyx loom goto` into Webster does not revive a dead batch.
Master's two dead-batch rungs (a `dead` batch with `recovery_retry: false` read at start, and a `recover-batch` digest with that status) write a `stuck_reason` that names the batch and, as its way forward, `lyx webster reset --to batch-start --batch <NN>` followed by `lyx loom goto Webster`, in fixed wording the stencil gives.
Master never runs that reset itself.
After that reset, the batch is recoverable: its recovery count and recovery start are cleared, and the next run's `recover-batch` or `begin-batch` for it is not refused as dead; where `reset` today leaves the dead status, it clears it.
Bound: only the operator's explicit reset re-opens the batch, and it discards the batch's commits and the run's tracked writes since the batch's start, as `batch-start` already does.
Rationale: the halt told the operator to re-enter Webster, which halted again at once on the same dead batch; the reset already exists and clears the counters, so the fix is naming it in stable wording.

## Constraints

- `PATTERN-completion-signal`: no new negative finalize; `D1`'s deadline exit goes through `classifyDeadlineExpiry`.
- `PATTERN-shuttle-provider-seam`: `D1` and `D2` stay provider-neutral in `shuttleengine`.
- `PATTERN-refusal-way-forward`: every changed or new refusal (`D4`, `D6`, `D7`'s Rename refusal) names its way forward and has its row in `refusal-spec.md` or the loom plan spec in the same commit, reached by a test.
- `PATTERN-sole-parsers` and `PATTERN-ref-shape-registry`: `D7` reads quarry answers and file constraints only; plan and handle parsing stays in `planparser`.
- `PATTERN-no-denied-recovery`: `D5` and `D8` name only commands the agents' settings allow.
- State compatibility: `D3`'s new `BatchState` field is `omitempty`, and an old `state.json` loads unchanged.

## Auto-mode assumptions

- This record faces code base `800a0e892`, per `ly:orch`'s instruction for this bench pair; read it with `git show 800a0e892:<path>`, since the checked-out tree is newer and was not reset.
- Self-picks, each logged in the support log's ledger: `D1` for every gated producer; `D4` non-zero for `dead` with `recovery_retry: true` too; `D7`'s free-tag cap of four; `D3`'s skip-with-warning for a missing recovery transcript; `D8`'s reset route over a goto revival.

## Open risks

- `D1`: a fork that leaves a never-ending background shell, or a shell whose completion shuttle never sees, holds Master for up to `master_timeout_min` with its work done, where today it ended at once; the wait marker's `background shells` label and the ended-shell record are the operator's signal.
- `D2`, `D5` and `D8` wording can only be judged by a live run; no `llm`-tier test covers whether a `sonnet[medium]` Master follows the fixer-fork line.

## Acceptance criteria

- A gated run started fresh, whose output files all exist at a turn end with only a background shell outstanding, keeps waiting and reaches the gate only at the turn end after the shell's completion.
- A gated run whose turn end lists only a shell a returned fork left behind keeps waiting the same way.
- A gated run whose shell never completes ends `OutcomeDone` at its deadline, with the gate run once at finalize and the shell in `Result.EndedShells`; Webster's summary and friction note say the shell's result was never read.
- A gated run's turn end with a fork outstanding still holds, and an attached or resumed gated run holds as before.
- An ungated run with every output file present and a shell outstanding still finishes `done` at that turn end.
- Webster's verify re-prompt opens `Gate findings recorded at <path>` and names the fixer fork and the never-fix rule; another gated producer's re-prompt, and an entry whose renderer returns empty text, send the default text.
- The verify findings text opens with the fixer-fork line; `verify-gate.yaml` is unchanged.
- A second recovery of a batch whose first recovery left uncommitted files lists those files under "Written by this run".
- A reset discards a tracked path only a recovery session of this run wrote, and keeps a path no recorded session wrote.
- A contract file last written by a recovery session stays uncleared, and its message names a recovery session as the last writer.
- A recorded recovery session whose transcript is missing leaves its paths foreign, logs a warning, and leaves every contract file uncleared.
- A recovery respawn and a re-begin keep every earlier recovery session id; a failed start and a pure attach add none; a `state.json` without the field loads with none.
- `recover-batch` on a terminal `failed`, `stuck` and `dead` batch exits non-zero with `ok: false`, the digest's `status` and `recovery_retry`, and no `batch_failed`, after the terminal state is saved and committed; on `done` and on a `--wait` running snapshot it exits 0.
- The rendered fork and recovery prompts name the always-allowed commands, the mutating-only fabric ban and the `go vet -tags` rule with packages listed one by one.
- `create-not-done` for a not-found target names the key and status; for an ambiguous target it names each candidate's file and constraint.
- The classifier reads as partitioned: `_linux.go` beside `_windows.go`; `unix` beside `_windows.go`; `//go:build tmux || llm` beside `//go:build !tmux && !llm`; `go1.21` beside `!go1.21`; `gc` beside `!gc`.
- The classifier reads as not partitioned: `_linux.go` beside a `unix` file; a name in package `x` and `x_test`; more than four free tags; an unreadable file; an unparsable constraint; a candidate under `//go:build linux && windows`.
- An Edit, a `Uses:`, a Create done-check and a Rename's new side over a partitioned member pass; a Rename's old side over one refuses with the Delete-plus-Create way forward; a Create target already partitioned is `create-already-exists`.
- After `lyx webster reset --to batch-start --batch <NN>` on a dead batch with no retry left, the batch is not refused as dead; Master's dead-batch `stuck_reason` names that reset.

## Notes for the plan writer

- `D1`'s hold sits in `expiredTurnEnd`/`countsAtOnce` (`internal/shuttleengine/wait.go`), and `waitmark.go` reads `awaitedShell` too; the corpus case `claudeengine/testdata/corpus/master-fork-running` describes the at-once reading.
- `D1` and `burler-multi-llm` both edit `internal/shuttleengine`'s turn-end handling: that task changes the held-turn-end notice path, this one the waiting turn end.
- `D2`'s line goes through the same send path `pane-inject-safety` reworks; keep it one line with the stable opening.
- `D3`'s session id is the shuttle run's own; the recovery record already holds `ShuttleRunDir`.
- `contracts/stencils/claims_test.go` pins stencil phrases `D5` and `D8` change.

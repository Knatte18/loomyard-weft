## Goal

Close six Webster faults that stop or mislead runs outside the run-exit audit (board entry items 1 to 6), so the next run takes the right path without operator help: background gate shells, Master fixing verify failures itself, recovery attribution and exit status, the recovery prompt's gates, and Create checks on build-constrained members.
For item 6, the Plan-time flag is informational: a build-partitioned glyph passes both Plan and record-batch, and Plan names its candidate files.

The design faces the code at base `800a0e892`; `1e79825ff` and everything after it count as not landed.

## Scope

In:

- `internal/websterengine`: Master's spawn spec (awaited shell prefixes, the verify gate entry's re-prompt), the recovery record and its uncommitted-paths attribution.
- `internal/shuttleengine`: one optional caller-supplied re-prompt on `GateEntry`.
- `internal/webstercli`: `recover-batch`'s exit status for a terminal digest.
- `internal/planglyph`: the build-partition predicate and its readers, and the `create-not-done` detail.
- Stencils: `contracts/stencils/webster/webster-template-master.md` (recover-batch rungs) and `contracts/stencils/webster/webster-prefix-recovery.md` (allowed gates, tmux steps).
- `contracts/specs/refusal-spec.md` rows, any spec that describes `recover-batch`'s exit, the package `doc.go` files of every package above, and `cmd/lyx/testdata/test-budget.yaml` where a new top-level test needs it.

Out:

- Quarry itself: it is an external module (`github.com/Knatte18/quarry`), so quarry honouring build constraints is a change in another repository.
- Recovery strands' own background shells: a recovery is an ungated run whose report is its completion contract, and the board entry names Master's wait.
- `#507`'s second half, a `Create` handle reusing a name the same card deletes everywhere.
- A Rename whose old side is a build-partitioned member: `renameDeclSource` stays found-only, since it derives the new declaration from exactly one old one.
- Reviving a dead batch through `lyx loom goto` (`#511`'s third point).
- Run-write attribution in `reset` and in the run-exit audit's contract evidence: both keep reading Master sessions only.
- The implementer stencil's `deviations` paragraph: owned by board entry `arbiter-fixes`.

## Decisions

### D1 — A background `lyx gate test` shell holds Master's turn end

Decision: Master's spawn declares `lyx gate test` as a second awaited shell prefix beside `masterAwaitedShellPrefix`.
A gated run's waiting turn end with every output file present then counts as an arrival only after every such shell has ended, whether the Stop payload or only the transcript reports it.
No shuttle change: `Spec.AwaitedShellPrefixes` and `countsAtOnce` already hold a turn end on an awaited shell.
Bound: the hold lasts at most until Master's `master_timeout_min` deadline or the liveness check ends the run; a shell still outstanding then comes back on `Result.EndedShells` and in the existing `webster-background-shell` note.
Matching is shuttle's prefix match on the shell's label, which is its command text, so only a command that starts with `lyx gate test` is held.

Rationale: the shell runs a test tier whose result the run would otherwise report as done.
At base no shell expires; the remaining path to a premature done is `countsAtOnce` admitting a fresh run's first arrival when every outstanding task is an unawaited shell.

### D2 — The verify gate's re-prompt names the fixer fork

Decision: `GateEntry` gains an optional re-prompt function from the findings path to one line; an entry without one keeps `gateRepromptText`.
Webster's verify gate entry sets it to a line that keeps the opening `Gate findings recorded at <path>`, says Master does not edit or commit for the findings, and tells it to spawn one fixer fork whose prompt is the file at `verify_fix_prompt_path`, then end its turn.
The Master stencil already states the no-edit rule right after its trigger line and keeps matching on the same opening, so it needs no change for D2.
Bound: the field changes only the text sent; the line goes through the same one-line send validation as every re-prompt before anything is typed, and a failing line is a returned gate error that ends the run as any gate error does.
The other gate entries Master's spawn carries (`deps.Gate`) keep the generic text.

Rationale: at base the only text Master sees at a verify failure is shuttle's generic "read it, fix every finding, and end your turn", which contradicts its stencil; the failure in `driver-context` followed the message.

### D3 — Recovery write attribution is by batch

Decision: each recovery spawn records its recovery session's id on the batch record, read from the shuttle run record once the spawn has come up, before the record is saved.
Every later fresh record of that batch (a respawn or a re-begin) carries forward the ids of all earlier recovery sessions, the prior record's own id included, the way `Recoveries` and `AuditWarnings` are carried.
`uncommittedPathsBlock` adds the parent write events of every recovery session the batch's current record names (its own id and the carried ones) to the run's write evidence, so a path the first recovery wrote lands under "Written by this run".
A re-entrant call that attaches to a live recovery changes no id; a start that fails, or a run record without a session id, records nothing.
Bound: only sessions on that batch's own record count, so another batch's recovery, or a session from before a `run --fresh` (which archives state.json), never turns a foreign path into the batch's own.
A path no recorded session wrote stays under "Not written by this run".

Rationale: at base `recordedSessions` reads only Master sessions, and a recovery strand is a separate session whose writes are therefore unattributable; the recovery prompt already treats "Written by this run" as the earlier attempt's work to keep.

### D4 — `recover-batch` exits non-zero unless the batch ended done

Decision: a terminal digest whose status is not `done` (`stuck`, or `dead` with any `dead_reason`) returns a failure envelope that carries the same digest fields, `recoveries` and `recovery_retry` the success envelope carries today, plus `batch_not_done: true`.
Its message names the way forward: with `recovery_retry: true`, re-run `lyx webster recover-batch NN` backgrounded; otherwise the run ends stuck and the way forward is the one the refusal spec already gives an exhausted batch.
A `done` digest and a running snapshot (`--wait` elapsed) keep exit 0, and a terminal `failed` batch keeps its existing non-zero `batch_failed` envelope.
The Master stencil's recover-batch rungs read the same fields from the failure envelope.

Rationale: an exit code is the first thing an operator, a script or a backgrounded-command notification reads, and 0 on a batch that did not finish reads as success.

### D5 — The recovery prompt names its allowed gates

Decision: `webster-prefix-recovery.md` gains a short section stating that the card `verify:` lines and the batch gate the implementer body lists, plus `lyx gate test` and `lyx loom lint-comments`, are always allowed and are not fabric commands.
It points at the body's sections instead of repeating their commands.
The same section says a step whose `--tags` names `tmux` or `llm` is not run by the recovery strand: it counts as no gate failure and no self-fix attempt, and the strand names each skipped step in its final reply.
Bound: a skipped tagged step is still compiled by the plan's `## verify:` (`go vet -tags tmux` and `-tags llm`) and its tmux tier runs at Publish through `publish_verify`; a batch whose gate holds only such steps reports `OK` on compilation alone until Publish.

Rationale: the cold strand read the sandbox rule's fabric ban as possibly covering the gates and could not run tmux-tagged steps; the board entry asks the prompt to leave those steps to verify.

### D6 — A build-partitioned ambiguous answer reads as multipart

Decision: planglyph gets one predicate over an ambiguous resolve answer: it is build-partitioned when every candidate sits in a different file under the worktree root and no two candidates' files can be built together.
A file's constraint is its `//go:build` expression together with the GOOS and GOARCH its file name implies, both as `go/build` applies them.
Two files can be built together when some GOOS/GOARCH pair from the toolchain's known list, with the tags the toolchain sets for that pair (its implied tags included, such as `unix`, and `linux` under `android`), plus some assignment of the remaining tags, satisfies both.
The predicate applies only when the plan's glyph language is Go.
Fail closed, so the answer is not partitioned: a candidate file that cannot be read, a constraint that does not parse, a file with no constraint, two candidates in one file, candidates that mix `_test.go` and non-test files, or more distinct free tags than a fixed cap.

A build-partitioned answer is read as a multipart answer whose symbols are its candidates, at every planglyph site that reads a resolve verdict:

- the status policy (`statusFindings`) passes it, with an informational finding naming the candidate files; it runs at Plan time and in `DetectDrift`'s post-repair pass at record-batch, and both callers are handed the worktree root;
- `answerSymbols` returns the candidates, so the delete-order check and the edit regions see each declaration;
- the plan-gate passes that skip an answer the status policy reports (caller-uncovered and the others in `planGatePass`) run on the candidates;
- `DoneChecks`: a Create target and a Rename's new side pass; a Delete target and a Rename's old side still block, since a partitioned answer still names live declarations;
- `create-already-exists` keeps blocking a Create whose target already resolves partitioned.

An ambiguous answer that is not partitioned keeps every current rule, the blocking glyph-ambiguous finding at Plan time included.
Bound: what newly passes is only an answer whose candidates cannot coexist in one build, and every check that reads declarations sees all of them.

Rationale: `#500`'s `tmuxAllowed` and the per-GOOS members of `#507` are correct Go that quarry answers ambiguous because it ignores build constraints.
One predicate read at every site keeps Plan time and record-batch in agreement.

### D7 — `create-not-done` says how it looked up the target

Decision: the `create-not-done` detail, and the `rename-not-done` new-side detail, name the resolve key looked up (a `plan:` handle's glyph half), quarry's status, and for `ambiguous` the candidate files with the reason the answer is not build-partitioned.
A `not_found` detail adds that the lookup covers the unit's exported and unexported declarations, its same-package `_test.go` files included, and that an external test package's declarations live under the unit `<unit>_test`.

Rationale: quarry already resolves unexported and same-package test-file identifiers `found`, so no resolver fix is needed; a detail with status and candidates tells a resolver limit from a missing symbol.

## Constraints

- `PATTERN-refusal-way-forward`: D4's new failure envelope and D7's changed details land with their rows in `contracts/specs/refusal-spec.md`'s webster section and a test reaching each.
- `PATTERN-no-denied-recovery`: no new message or stencil line names a denied form; D4's ways forward name `lyx webster` verbs only.
- `PATTERN-shuttle-provider-seam`: D1 and D2 add no provider knowledge to shuttle; the prefix and the re-prompt are caller data.
- `PATTERN-stencil-ownership`: stencil edits go to `contracts/stencils/webster/`; D2's re-prompt is a one-line Go-built message like shuttle's own `gateRepromptText`, and it names the prompt file, not a stencil heading.
- `PATTERN-told-geometry`: D6 reads candidate files under the worktree root its callers are handed; `statusFindings` gains that root from its callers.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing tables (`countsAtOnce`, gate re-prompt, `DoneChecks`, status policy, plan-gate passes, recovery prompt, recover-batch CLI) before adding top-level tests; a test touching quarry runs at the tier `donecheck_resolve_test.go` runs at.
- Docs in the same commit: `internal/websterengine/doc.go` ("Background shells in Master's wait", the recovery prompt paragraph, the verify gate re-prompt), `internal/shuttleengine/doc.go` (the gate re-prompt), `internal/planglyph/doc.go` (the check list and the partition rule), `internal/webstercli`'s package doc where it describes exits, and `recover-batch`'s `Long` text.
- Board entry `arbiter-fixes` is in flight and may edit `contracts/stencils/webster/` (Master's digest and summary rules); the two runs may meet at Merge-In on `webster-template-master.md`. No dependency either way.
- No new PATTERN entry: every change applies an existing rule.

## Auto-mode assumptions

- A1: D1 holds the shell rather than stopping it (auto-pick).
- A2: D2 fixes the message rather than adding a verb (auto-pick).
- A3: D3 attributes by batch rather than starting from a clean tree (auto-pick).
- A4: D4 counts a `dead` digest with `recovery_retry: true` as not done, so it exits non-zero too.
- A5: the board's "exited 0 while the batch ended failed and terminal" is read as a terminal `stuck` or `dead` digest, the only terminal outcomes that exit 0 at base; a terminal `failed` already exits non-zero through `batch_failed`.
- A6: the board's "the plan validator flags such a glyph at Plan time" is read as D6's informational finding plus the unchanged blocking glyph-ambiguous for a non-partitioned answer; a Create's target does not exist at Plan time.
- A7: `#511`'s targets are taken to have failed through D6's cause, from the landed shape of `cmd/testtiming`.
- A8: D5's tmux skip follows the board entry; the record does not establish why the recovery strand could not run tmux-tagged steps when implementer forks run them through `lyx gate test`.

## Open risks

- R1: A gate command not starting with `lyx gate test` (one prefixed by `cd` or `timeout`) is not held by D1; nothing in the settings refuses those forms.
- R2: D1 lets a long tier, or a shell whose completion the transcript never shows, hold Master up to `master_timeout_min`, which then ends the run on its deadline.
- R3: D1 also delays a `paused` or `stuck` outcome Master writes while a gate shell runs, until the shell ends; accepted, since the shell's result is the point.
- R4: D6 depends on `go/build`'s tag implications and known GOOS/GOARCH list; a value outside them makes a pair not partitioned (fail closed).
- R5: D3 relies on the recovery session's transcript being readable when the next recovery spawns; an unreadable transcript is an audit error today and stays one.

## Acceptance criteria

- A Master run whose turn end has every output file present and an outstanding background shell labelled `lyx gate test --tags integration ./... 2>&1 | tail` is not done at that turn end, for a shell reported by the Stop payload and for one reported only by the transcript; a later turn end after the shell ended is the arrival.
- A verify-gate failure sends Master one line that opens with `Gate findings recorded at`, names the fixer fork and its prompt file, and says Master does not edit or commit; a gate entry with no re-prompt function sends the generic line; a malformed line is a gate error raised before anything is typed.
- A second recovery of a batch whose first recovery left uncommitted files, including after a Webster restart stopped that first recovery, lists those files under "Written by this run"; a file no recorded session of that batch wrote stays under "Not written by this run".
- A failed start records no session id; an attach to a live recovery adds and drops none; the recorded ids survive a respawn and a re-begin without duplicates.
- `recover-batch` exits non-zero with `batch_not_done: true`, the digest fields, `recoveries` and `recovery_retry` for a terminal `stuck` digest and for a terminal `dead` digest with either `recovery_retry` value; it exits 0 for `done` and for a running snapshot; a terminal `failed` batch still exits non-zero with `batch_failed`.
- The rendered recovery prompt names `lyx gate test`, `lyx loom lint-comments`, the card `verify:` lines and the batch gate as allowed, and says `tmux`- and `llm`-tagged steps are skipped and named in the final reply.
- `DoneChecks` passes a Create whose target is declared once in a `//go:build tmux || llm` file and once in a `//go:build !tmux && !llm` file, and one declared in `x_linux.go` and in a `//go:build !linux` file.
- `DoneChecks` keeps `create-not-done` for an ambiguous answer whose files can build together (`//go:build linux` beside `//go:build unix`; `x_linux.go` beside `//go:build android`), whose candidate file has no constraint, whose two candidates share a file, or whose candidates mix a `_test.go` and a non-test file.
- `DoneChecks` keeps `delete-not-done` for a Delete whose target is build-partitioned.
- At Plan time, a build-partitioned Edit glyph gets an informational finding naming the candidate files, a non-partitioned ambiguous glyph keeps the blocking glyph-ambiguous, and a Delete of a build-partitioned member that a later card still references is refused by the delete-order check.
- A Create target that is an unexported identifier declared in a same-package `_test.go` file passes `DoneChecks`.
- A `create-not-done` finding's detail names the resolve key, quarry's status and, for ambiguous, the candidate files.

## Notes for the plan writer

- Master's spawn spec and verify gate entry are built together in `internal/websterengine/runlevel.go`, next to `NewVerifyGate`.
- `recoverSpawn` in `internal/websterengine/recoverbatch.go` builds the fresh `BatchState` and already reads the recovery's run state after the start.
- The terminal-digest envelope is built near the end of `internal/webstercli/recoverbatch.go`'s `RunE`.
- planglyph's ambiguous readers: `resolve.go`, `donecheck.go`, `create.go`, `deleteorder.go`, `plangate.go`, `drift.go` and `handle.go`.

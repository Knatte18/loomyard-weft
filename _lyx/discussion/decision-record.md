# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close the six Webster faults board entry `twb-fta-2` lists, which stop or mislead runs outside the run-exit audit.
The record faces the code at base `800a0e892`.

## Scope

In:

1. Master's turn end held by a running `lyx gate test` shell; the end-of-run shell warning names a gate shell's result as never read.
2. The verify gate's re-prompt to Master names the fixer fork and forbids fixing in Master's own session.
3. Recovery strands' writes count as the run's own; `recover-batch` exits non-zero when the batch ends terminal without a done recovery.
4. The recovery prompt names the card and batch gates as allowed and leaves `tmux`-tagged card steps to verify.
5. The Create and Rename-new done-check refusals name the resolve key, quarry's status and, for an ambiguous answer, the candidate files.
6. A Go member whose ambiguous candidates sit in files with pairwise disjoint build constraints reads as one multipart member in every planglyph pass, at Plan time and at record-batch alike.

Out:

- Teaching quarry itself (`github.com/Knatte18/quarry`, an external module at v0.2.1) build constraints; the fold lives in lyx.
- #507's second half: a `plan:` Create that reuses a name the same card deletes still refuses `create-already-exists`. The board's expected outcome for item 6 names only the ambiguity.
- #511's question whether `lyx loom goto` into Webster revives a dead batch.
- Holding an ungated run's turn end, such as a recovery strand's, on a gate shell: no incident shows it, and a recovery's batch is re-checked by the post-batch pass and the plan-level verify.
- Changing the implementer fork's prompt for the `tmux` tier; item 4 changes the recovery prompt only.
- Build constraints for any glyph language but Go.

## Decisions

### D1. Master awaits its gate shells

Decision: Master's spawn declares a second awaited shell prefix, `lyx gate test`, beside `masterAwaitedShellPrefix`.
A gated Master turn end with every output file present and a shell whose label starts with `lyx gate test` outstanding no longer counts at once; it waits for the shell's completion notification, bounded by the run's `Timeout` and the liveness check as an awaited `recover-batch` shell already is.
Shuttle's rule (`countsAtOnce`) does not change: the prefix is caller data on `Spec.AwaitedShellPrefixes`.

Rationale: at base no shell expires, and the remaining path that cut the gate shell off is the first-arrival rule: a fresh gated run whose only outstanding tasks are unawaited shells finishes Done at once when its output files exist.
The shells in #505, #509 and #517 were `lyx gate test --tags integration …` runs, and the Claude engine labels a shell by its command text, so a prefix match catches them, piped tails included.
Lyx cannot stop a shell (the provider reports no pid), so holding is the one board option lyx can carry out.

Bound: the hold lets nothing through that was refused before; it delays a Done until the shell's notification, capped by the run's `Timeout`.

### D2. A gate shell outstanding at the run's end is named as unread

Decision: the per-shell wording of Webster's `webster-background-shell` friction note and its `RunResult.Warnings` entry gains one clause for a shell whose label starts with `lyx gate test`: its result was never read by the run, so the run's outcome says nothing about that gate.
Every other shell's wording is unchanged.

Rationale: after D1 such a shell can still be outstanding when the run ends on its deadline, liveness, a died error or a spelling the prefix misses.
The existing note and warning already list each shell and say lyx did not stop it, so one clause there keeps the outcome from reading as "tests passed".

### D3. The verify gate's re-prompt points at the fixer fork

Decision: `shuttleengine.GateEntry` gains an optional caller-supplied re-prompt text builder, given the findings path and returning the one-line text the wait loop sends after a failed attempt.
An entry without one keeps `gateRepromptText`'s text.
Webster's `verify` entry (`NewVerifyGate`) supplies one that names the findings file, says to spawn exactly one fixer fork whose prompt is the verify-fix prompt file Go rendered at run entry (named by its path), and says never to edit, stage or commit in Master's own session.
The text keeps the one-line shape `validateSendText` requires.

Rationale: the message Master reads at a verify failure is shuttle's generic "read it, fix every finding, and end your turn", which contradicts its stencil's "Do NOT localize or fix the failure yourself"; on attempt 3 of a long session the message won.
The rule now sits in the one message Master reads at the failure, and it names the prompt file by path, so no stencil heading is quoted from Go.

Bound: the builder changes text only; it cannot change the entry's attempt budget, its verdict, or which turn boundary sends it.

### D4. Recovery strand writes are the run's own

Decision: each recovery spawn records its shuttle session id on the batch record, in the same write that records its strand, as a list carried across a respawn and a re-begin the way `Recoveries` is carried; an attach records nothing new.
`loadRunWrites` audits those sessions too and returns their writes as a third group beside `Master` and `Forks`.
`recordedSessions` does not change, so the run-exit fork audit and the contract-file evidence (`contractevidence.go`, which reads Master's writes) never see a recovery session.
The recovery prompt's uncommitted-path classing (`recoverbatch.go`) and a reset's own-path set (`reset.go`) count the new group as the run's writes.

Rationale: at base `recordedSessions` lists only Master sessions, so a recovery strand's writes are never evidence, and the second recovery of #522 saw the first one's 19 files under "Not written by this run".
Attributing by batch keeps webster fabric-blind (`# engine/cli split`).

Bound: a path counts as the run's own only when a recorded recovery session's transcript shows a successful write to it; a file changed only through a Bash command stays foreign, as a fork's does today.
A reset may now discard a recovery's uncommitted writes as it already discards Master's and the forks'; its existing refusals still bound it: every target but `report-head` refuses while the run lock is held, and `report-head` refuses while a recovery strand of the batch is live in reed.

### D5. `recover-batch` exits non-zero when the batch ends terminal

Decision: a terminal digest that leaves the batch terminal without a done recovery, `status: stuck`, or `status: dead` with `recovery_retry: false`, returns a failure envelope and a non-zero exit.
The envelope carries the same digest fields, `recoveries` and `recovery_retry` as today's success envelope, and an error naming the batch, the status and the way forward: the run ends stuck on that batch.
A `done` digest, a `dead` digest with `recovery_retry: true` and the `status: running` snapshot keep exit 0.
A re-entrant call that returns such a terminal digest exits the same way.
The new refusal gets its row in `contracts/specs/refusal-spec.md` and a test, and, like every `ok: false` webster envelope, writes a `webster-refusal-recover-batch` friction note, which is wanted: a terminal batch is a failure for reflection.
Master's stencil rungs for a terminal `stuck` and a `dead` with `recovery_retry: false` say the call exits non-zero and that the rung is still read from the envelope's fields.

Rationale: #522 saw exit 0 on a batch that ended failed and terminal, which reads as success to anything keyed on the exit code.
A retryable dead recovery is not terminal, and Master's ladder retries it, so it stays a success.
Keeping every field lets Master's rungs, keyed on `status` and `recovery_retry`, read the envelope unchanged.

### D6. The recovery prompt names its allowed gates

Decision: the recovery prefix stencil (`webster-prefix-recovery.md`) gains a short section stating that every card-gate and batch-gate step it lists (`lyx gate test` at whatever tags the step names, and `lyx loom lint-comments`) is always allowed and is neither a `lyx fabric` nor a `lyx webster` verb.
It also states that a card `verify:` step carrying the `tmux` tag is not run by the recovery, does not count as a failure, and is left to the verify Publish runs.

Rationale: the quick-recipe recovery could not tell whether its gates fell under the bans on `lyx fabric` and `lyx webster`, and could only compile-check its `tmux`-tagged steps.
The plan spec already leaves the `tmux` tier to Publish (`loom-plan-spec.md`, `## verify:`), and the stencil points at that split.

Bound: the skip covers a `verify:` step tagged `tmux` only; every other step stays mandatory and counts under the self-fix bound.

### D7. Create and Rename-new refusals say how the target was looked up

Decision: the `create-not-done` and the new-side `rename-not-done` details name the resolve key (a `plan:` handle's member-glyph half), quarry's status, and for `ambiguous` each candidate's file with a note that their build constraints overlap or could not be read.
A `not_found` detail names the unit looked in and that a member of an external test package (`package x_test`) resolves under the unit `<dir>_test`.

Rationale: quarry v0.2.1, the version at base, resolves unexported members and members of same-package `_test.go` files (checked on the pair's tree, same quarry version), so the check needs no resolver fix; what was missing is the status, which a bare "still does not resolve" hid.

### D8. Disjoint build-constrained candidates fold into one multipart member

Decision: a planglyph helper rewrites an `ambiguous` answer into a `multipart` answer whose symbols are the candidates, when the plan's glyph language is Go and every pair of candidate files has disjoint build constraints.
Every plan-reading pass that resolves through `resolveTargets` (the status policy, Create checks, DoneChecks, delete-order, drift's revalidation, the plan gate's member checks) applies it to its answers, given the plan's language.
`planglyph.Resolve`, which serves `lyx quarry resolve`, does not apply it, so agents keep quarry's raw answer.
A file's constraint is its `//go:build` (or legacy `// +build`) expression conjoined with the GOOS and GOARCH its file name implies (`_linux.go`, `_windows_amd64.go`).
Two files are disjoint when no assignment of their tags satisfies both, where at most one known GOOS and at most one known GOARCH is true and the `unix` tag follows the GOOS; every other tag, `ignore` included, is free.
A candidate file with no constraint, or one that cannot be read or parsed, keeps the answer `ambiguous`.

Rationale: one fold applied by every pass gives the plan validator and DoneChecks the same verdict, so a glyph that passes Plan cannot fail record-batch on ambiguity alone.
`multipart` already means "one member legitimately declared in several places": a Create is done, a Delete is not done while any declaration remains, an Edit or `Uses:` glyph resolves, and a Rename's old side still needs `found`.
At Plan time an Edit, Delete or `Uses:` glyph of a per-GOOS member (`internal/proc#StartTime`, `internal/gatecli#runChild`) stops refusing `glyph-ambiguous`, and one with overlapping candidates still refuses there; that is the Plan-time flag the board asks for.
A Create has no Plan-time flag, because its declarations do not exist before the card runs.
`BindHandles` binds a `plan:` handle from the git delta's created symbol ids, so two declarations of one id already bind it once.

Bound: the fold lets through only an answer whose candidates cannot be compiled into one binary together; candidates that can coexist, which the compiler would reject as a redeclaration, stay `ambiguous` and blocking.

## Constraints

- `PATTERN-shuttle-provider-seam`: the gate-shell prefix and the re-prompt text are webster's caller data; shuttle gains a provider-neutral builder field.
- `PATTERN-refusal-way-forward`: D5's refusal lands with its `refusal-spec.md` row and a test; D7 updates the `create-not-done` row's wording if the spec quotes it.
- `PATTERN-no-denied-recovery`: D5 and D6 name no command the agents' settings deny.
- `PATTERN-glyph-conversion-chokepoint`: D7's unit name and D8's rebuilt answer derive units and paths through `glyph` only.
- Docs in the same commit: `internal/websterengine/doc.go` (`# Background shells in Master's wait`, `# The verify-gate fixer fork`), `internal/shuttleengine/doc.go` (gate re-prompt), `internal/planglyph/doc.go`, `recover-batch`'s `Long` text, and the plan spec's glyph-resolution text for D8.
- A package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- A1. Gate shells start with `lyx gate test`, as the reported ones and the stencils' commands do (D1).
- Every decision D1–D8 is an auto-pick; the support log's question ledger records each.

## Open risks

- R1. A gate shell whose command does not start with `lyx gate test` (a leading `cd`, an env assignment, a full binary path), or one under an engine that labels shells by something other than their command, is not held; D2's warning is then the only signal.
- R2. A gate shell that hangs, or whose completion notification never matches its launch, holds Master until the run's `Timeout`.
- R3. A recovery session whose transcript is gone (another machine, a pruned projects directory) yields no write evidence, so its files fall back to foreign as at base.
- R4. D8's tag model knows the GOOS and GOARCH lists compiled into lyx; a newer toolchain's GOOS reads as a free tag, which can only make two files look overlapping, never wrongly disjoint.
- R5. `arbiter-fixes` changes the implementer body's `deviations` paragraph, which the recovery prompt composes with D6's prefix, and may change what the `deviations` digest field D5 carries means; whichever lands second rebases on the other's text.
- R6. `weaver` plans the recovery strand as the normal implementer path, which would make D6's recovery-only `tmux` skip the default; that entry decides whether it carries over.

## Acceptance criteria

- A gated Master run, started fresh, whose turn end has every output file present and a `lyx gate test --tags integration …` shell outstanding keeps waiting, and finishes on the turn end after the shell's notification.
- The same turn end with only an unprefixed shell outstanding still finishes at once, as at base.
- A run ending with a `lyx gate test` shell outstanding records, in both the friction note and `RunResult.Warnings`, that the shell's result was never read.
- After a failed verify attempt, the text sent to Master names the findings file and the verify-fix prompt path and forbids fixing in Master's session; an entry with no builder still sends the generic text.
- A second recovery of a batch lists the first recovery's successfully written, still-uncommitted files under "Written by this run", including after a respawn that stopped the first strand.
- A re-entrant `recover-batch` attach records no duplicate session id.
- A path neither Master, a fork nor a recovery session wrote stays under "Not written by this run".
- A recovery session's writes never appear as Master's writes in the run-exit audit or the contract-file evidence.
- A reset's own-path set includes a recovery session's written tracked paths.
- `recover-batch` exits non-zero with `status`, `recovery_retry` and `recoveries` in the envelope for a terminal `stuck` digest and a `dead` digest with `recovery_retry: false`, and writes the refusal friction note; it exits 0 for `done`, for `dead` with `recovery_retry: true` and for a `running` snapshot.
- The rendered recovery prompt names the card and batch gates as allowed and the `tmux`-tagged `verify:` steps as left to verify.
- A `create-not-done` finding for an ambiguous answer names the key, `ambiguous` and each candidate file; one for `not_found` names the unit looked in.
- A batch whose card creates `plan:<pkg>#<name>` once in a `//go:build tmux || llm` file and once in a `//go:build !tmux && !llm` file passes record-batch end to end: handle binding, drift, the scope guard and DoneChecks.
- A member declared in `x_linux.go` and `x_windows.go` passes DoneChecks as a Create, and an Edit of it passes the plan validator.
- Candidates in two files whose constraints overlap, where one file has none, or where one carries only an unparsable constraint stay `ambiguous`: blocking at Plan time for an Edit and at record-batch for a Create.
- A Delete of a folded member stays `delete-not-done` while any declaration remains.
- `lyx quarry resolve` on a per-GOOS member still answers `ambiguous`.

## Notes for the plan writer

- Shuttle: `Spec.AwaitedShellPrefixes`, `countsAtOnce` and `awaitedLabel` in `internal/shuttleengine/wait.go`; `gateRepromptText` in `gate.go`.
- Webster: `masterAwaitedShellPrefix` and the spawn spec in `runlevel.go`; `NewVerifyGate`; `recordedSessions` and `loadRunWrites` in `runwrites.go`; `uncommittedPathsBlock` and `recoverSpawn` in `recoverbatch.go`; `BatchState` in `state.go`.
- CLI: the terminal-digest branch of `internal/webstercli/recoverbatch.go`; `noteRefusals` in `refusalnote.go`.
- Planglyph: `resolveTargets` and `Resolve` in `repo.go`, `doneCheckVerdicts` in `donecheck.go`, `candidateList` in `resolve.go`, `BindHandles` in `handle.go`.
- `go/build/constraint` parses `//go:build` and `// +build` lines; `go/build`'s file-name rules give the implied GOOS and GOARCH.

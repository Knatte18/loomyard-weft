## Goal

Close the Webster faults outside the audit that stop or mislead runs:
a background gate shell cut off at a turn end, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that does not name its allowed gates, and Create checks that cannot pass for unexported, test-file or build-tagged members.

## Scope

In:

- Master's wait for a background `lyx gate` shell (#505, #509, #517).
- The text Master reads when the verify gate fails.
- Recovery write attribution by batch, `recover-batch`'s exit code on a failed batch, and the recovery prompt's gate wording (#522).
- The `create-not-done` refusal's lookup report and the cause of the #511 refusals.
- `DoneChecks` and the plan-time resolve status policy for members declared once per mutually exclusive build-constraint set or once per GOOS (#500, #507).

Out:

- Changing the quarry module (`github.com/Knatte18/quarry`, pinned in `go.mod`): its resolver stays build-constraint-blind, and the loomyard side reads constraints.
- Killing a shell from lyx: shuttle has no pid for a provider shell.
- Fork turn-end noise (board entry `fork-wait-noise`) and fabric audit classifier work (`audit-reader-globs`).
- A new `lyx webster` verb that spawns the verify fixer.

## Decisions

1. **A running gate shell holds Master's turn end.**
   Decision: Master's shuttle spec adds the `lyx gate` command prefix to `AwaitedShellPrefixes` beside `masterAwaitedShellPrefix`, so a backgrounded `lyx gate test` shell is waited on like a fork.
   At base the cutoff is the gated fresh-arrival rule: a waiting turn end with every output file present finishes Done through the gate when every outstanding task is an unawaited shell.
   An awaited shell holds that turn end in every case, so the hold changes this path.
   Rationale: holding is the only outcome the code can deliver, since lyx cannot stop a provider shell, and the `AwaitedShellPrefixes` seam already exists for the recover-batch shell.
   Bound: the prefix matches by `HasPrefix` on the shell's label, so a shell labelled with a leading `cd … &&` or an env prefix is not held; the card reads the label shape the provider reports for the incident command and matches that.
   A held gate shell ends the wait only by its output files, the run's deadline or the liveness check, so a hung gate holds Master until `RunTimeoutMin`; no shorter bound is wanted.
   Every other unawaited shell keeps today's path: the run may finish `done`, and `Result.EndedShells`, the run warning, the `webster-background-shell` friction note and summary.md's shell section record it.
   An awaited gate shell outstanding at a timeout or died exit is recorded the same way, so the outcome never reads as "tests passed".

2. **The rule that Master never fixes a verify failure sits in the text Master reads.**
   Decision: the verify gate's re-prompt for Master states that Master spawns one fixer fork from the named prompt file and edits nothing itself.
   At base the re-prompt is shuttle's generic `gateRepromptText`, which tells the writer to fix every finding, so it contradicts the Master stencil; the fix supplies Webster's own text through the gate's carried text rather than adding a Webster sentence to the generic string.
   The stencil's gate-failure section keeps its numbered steps.
   Rationale: the incident was a sonnet[medium] Master acting on the findings it had just read, under a message that told it to fix them.
   Bound: this is guidance, not a guard; the run-exit parent-write audit stays the backstop and `accept-audit` stays the operator's recovery.

3. **Recovery write attribution is by batch.**
   Decision: the batch record gains a list of the Claude session ids of its recovery strands, written when each recovery is spawned, under the state-mutation lease, saved with the state, deduplicated.
   `loadRunWrites` audits those sessions beside the Master sessions, so their paths count as "Written by this run" in a later recovery's uncommitted-paths block.
   Rationale: at base nothing records a recovery strand's session id, so the second recovery in `quick-recipe` saw the first recovery's 19 files under "Not written by this run" and blocked; keeping the first attempt's files as the starting point is what the prompt already says for the run's own writes.
   Bound: a path no session of this batch's run wrote, a path another batch's recovery wrote, and a path the evidence cannot attribute stay foreign and untouched.
   A `state.json` without the list loads with it empty.
   Plan confirms that `AuditForks` over a recovery session returns that strand's own writes.

4. **`recover-batch` exits non-zero when the batch ends failed and terminal.**
   Decision: a recovery digest whose status is failed returns the `batch_failed` error envelope with the batch name and warnings, the same shape a post-batch-check rejection uses, after the failed state is saved and the fabric sync done.
   The error names the Master stencil's terminal rung for a failed recovery (never another `recover-batch`), and its spec row lands with it.
   Rationale: the verb's help says a rejected recovery exits non-zero; the observed exit 0 is the digest-failed path, which returns `output.Ok`.
   Bound: the premise that this path is the exit-0 one is checked at Plan time; the Master stencil's ladder wording and its `claims_test.go` pins stay consistent with the envelope.

5. **The recovery prompt names the always-allowed gates.**
   Decision: `webster-prefix-recovery.md` says `lyx gate test` and `lyx loom lint-comments` are always allowed to the recovery strand, as the card and batch gates, and that tmux-tagged steps are compile-checked only and left to verify.
   Rationale: the recovery doubted both commands against the fabric-command ban and could only guess at tmux-tagged steps.
   Bound: the sentence adds no command beyond those the implementer body already allows, and does not lift the ban on `lyx webster` or mutating `lyx fabric` verbs.

6. **A done-check refusal reports how the target was looked up.**
   Decision: every `DoneChecks` finding for a Create, Delete or Rename target names the resolve key and the status quarry returned, with the candidate files for `ambiguous` and the `unit` answer for `not_found`.
   Rationale: the #511 refusals read as "still does not resolve" for both a missing symbol and a resolver limit.
   Bound: the report changes message text only; no check passes or fails differently because of it.

7. **#511 is reproduced before it is explained.**
   Decision: the Plan fixes the cause the reproduction shows, trying `rusageStats` and `tmuxAllowed` through `Resolve` over a tree holding the declaration in a non-test file, a `_test.go` file and a build-tagged pair.
   `tmuxAllowed` is itself a build-tagged pair, so decision 8 may be the whole cause.
   A test-file or unexported limit, if shown, is fixed on the loomyard side; one in quarry that the loomyard side cannot absorb stays a refusal carrying decision 6's lookup report, and the record names it as a quarry limit.
   Bound: no limit is worked around by passing the check.

8. **An ambiguous answer whose candidates have mutually exclusive build constraints counts as resolved.**
   Decision: planglyph decides disjointness from each candidate file's own constraints, a GOOS or GOARCH filename suffix and any `//go:build` line, read through `go/build/constraint`; the `unix` tag is true exactly when a unix GOOS is.
   An answer is accepted only when it has at least two candidates, all in one unit, and every pair is disjoint: no assignment of the tags involved, with at most one GOOS and one GOARCH true, satisfies both.
   `DoneChecks` treats such an answer like `found` for Create and the new side of Rename, and the plan-time status policy raises no blocking `glyph-ambiguous` for it, leaving an informational finding that lists the variant files.
   A plan creates every variant of a member in one card; a Create of a further variant of a member that already resolves stays `create-already-exists`.
   Rationale: the resolver deliberately reports every build-tagged declaration as a candidate, so without this a Create or Edit of `tmuxAllowed`, `StartTime` or `runChild` can never pass; the plan-time downgrade is needed because Edit hits `glyph-ambiguous` there, and the informational finding is the flag the board entry asks for.
   Bound: a single candidate (a unit collision), candidates in different units, a candidate with no constraint, two candidates in one file, or two whose constraints overlap stay `ambiguous` and blocking.
   A malformed constraint or an unreadable candidate file blocks, wrapping `ErrQuarryUnavailable` for the read.
   A Delete or the old side of a Rename still treats any candidate as still existing.

## Constraints

- `PATTERN-refusal-way-forward`: the changed `batch_failed` envelope and the done-check report name a way forward, land their spec row (`contracts/specs/loom-plan-spec.md` for the plan-time finding) and carry a test.
- `PATTERN-no-denied-recovery`: the recovery prompt names no command the agents' settings deny, so `lyx gate test` and `lyx loom lint-comments` must pass the settings the recovery strand runs under.
- `PATTERN-stencil-ownership`: prompt and stencil edits live in `contracts/stencils/webster/`, read at call time, never embedded; the Go-authored gate re-prompt text of decision 2 stays Go-authored.
- `PATTERN-shuttle-provider-seam`: the awaited prefix is caller data on the spec; no provider specific moves out of `claudeengine`.
- `PATTERN-completion-signal`: `allOutputFilesExist` is still consulted first; if the hold needs a new call site, the pattern's scan list updates in the same card.
- `PATTERN-told-geometry`: candidate files are read from the worktree root planglyph is handed, with no cwd resolution; `PATTERN-leaf-packages` import sets are checked for the added `go/build/constraint` import.
- `PATTERN-test-economy`, `PATTERN-test-speed`: new tests pin public behavior at the lowest fitting tier; a card pushing a package past its `cmd/lyx/testdata/test-budget.yaml` row lists that file under `Edit:`.
- `PATTERN-comment-line-breaks` and CLAUDE.md's docs rule: the module docs the changes touch (`internal/websterengine/doc.go`, `internal/shuttleengine/doc.go` whose awaited-shell wording changes, `internal/planglyph/doc.go`) update in the same commit.
- Other board entries edit neighbouring text: `fork-wait-noise` edits the Master template and its claims tests, so the `lyx gate` prefix must keep matching the command text Master sees; `proc-child-lifetime` moves `lyx gate test`'s child-lifetime code, which this work leaves alone.

## Auto-mode assumptions

- The `lyx gate` prefix covers every gate tier, since any tier is a test result.
- Disjointness is judged on filename and `//go:build` constraints only; `+build` lines and cgo-dependent tags are not special-cased.
- The code is read at base `800a0e892`, as the board entry's bench note requires.

## Open risks

- A hung `lyx gate` shell now holds Master up to the run timeout; the wait marker and the `background_shell_wait_min` log are the only early signal.
- Reading each ambiguous candidate's file adds disk reads to every DoneChecks and plan-time resolve pass that sees an ambiguous answer.
- Edit of a variant member names several declarations; the implementer must edit each variant the card intends, and no check verifies that.
- If a recovery session id is never captured, attribution falls back to today's foreign classification, which is safe but blocks as in #522.

## Acceptance criteria

- A gated Master turn end with only a `lyx gate test` background shell outstanding and its output files present is held, not finished `done`; the label shape the provider reports for the incident command matches the prefix; a non-gate unawaited shell still finishes `done` and is recorded in `EndedShells`, the warning and the friction note.
- A held gate shell outstanding when the run times out or dies is recorded in `EndedShells` and the run's warning and does not read as passed.
- The verify gate's re-prompt for Master carries the spawn-one-fixer-fork instruction and the fixer prompt path, and no instruction to fix the findings itself.
- A second recovery of one batch lists the first recovery's uncommitted files under "Written by this run", including after a restart that stopped the first recovery uncommitted; a path no session of the batch wrote, or another batch's recovery wrote, still lands under "Not written by this run"; a batch with one recovery, and a `state.json` without the session list, behave as today.
- `recover-batch` on a recovery whose digest is failed exits non-zero with `batch_failed`, the batch name and warnings, after `state.json` holds the terminal failed batch; a re-run spawns nothing.
- The recovery prompt names `lyx gate test` and `lyx loom lint-comments` as allowed and leaves tmux-tagged steps to verify, and both commands pass the recovery strand's settings.
- A `create-not-done` finding names the resolve key and the returned status, and lists candidate files for `ambiguous`.
- A Create of an unexported member, and of a member declared in a `_test.go` file, resolves or is refused with a report that tells a missing symbol from a resolver limit.
- A Create or Edit of a member declared in two files with disjoint constraints (`//go:build tmux` against `//go:build !tmux`; `_linux.go` against `_windows.go`; a `unix` file against a `windows` file) passes `DoneChecks` and raises no blocking plan-time finding, with an informational finding listing the files; three disjoint candidates pass the same way.
- A member declared twice in one file, in two files with overlapping constraints, in one constrained and one unconstrained file, in different units, or a unit-collision ambiguous with a single match stays `ambiguous` and blocking.
- A malformed `//go:build` line or an unreadable candidate file blocks.
- A Delete whose target resolves ambiguous across disjoint-constraint files still fails `delete-not-done`.

## Notes for the plan writer

- Starting points: `internal/shuttleengine` (`AwaitedShellPrefixes`, `awaitedLabel` in `wait.go`, `gateRepromptText` in `gate.go`), `internal/websterengine` (`runlevel.go` Master spec, `runwrites.go`, `state.go`, `recoverbatch.go`), `internal/webstercli/recoverbatch.go`, `internal/planglyph` (`donecheck.go`, `resolve.go`, `create.go`) and `contracts/stencils/webster/`.
- Compute fresh which other arms read an ambiguous answer and need the same disjointness helper: `create.go`, `deleteorder.go`, `plangate.go`, `handle.go`, `drift.go`.

# Decision record — Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close six Webster faults outside the audit that stop or mislead runs: a background gate shell cut off when Master's run finishes, Master fixing a verify failure itself, a second recovery treating the first recovery's work as foreign, `recover-batch` exiting 0 on a failed recovery, a recovery prompt that leaves its gates in doubt, and Create and Edit checks that can never pass for a member declared once per build-constraint set.

The record faces the code at base `800a0e892`.

## Scope

In:

- `internal/shuttleengine`: the awaited-shell match and a per-entry gate re-prompt text.
- `internal/websterengine` and `contracts/stencils/webster/webster-template-master.md`: Master's spawn spec, verify-gate re-prompt, shell warning, recovery write attribution and failure ladder.
- `internal/webstercli`: the `recover-batch` exit code.
- `contracts/stencils/webster/webster-body-implementer.md`: the gate wording, which the fork and the recovery prompt share, so forks get it too.
- `internal/planglyph`: the build-variant reading of an ambiguous answer and the `create-not-done` detail.
- The docs each change touches: the packages' `doc.go`, `contracts/specs/refusal-spec.md`'s webster section, and the stencil claims tests.

Out:

- Changing quarry, an external module (`github.com/Knatte18/quarry`): lyx reads its answers as they are.
- Stopping a background shell: see Decision 1.
- How forks wait for their gates, in-turn or backgrounded: board entry `fork-wait-noise` owns that.
- Reviving a dead batch through `lyx loom goto` (#511's third point): not on this entry.
- A Create handle reusing a name the same card deletes (#507's first option): Decision 7 makes glyph Edit and Delete of a per-GOOS member work, #507's second option.
- Resolving an external-test-package member under its package's own unit: quarry answers it under the `<dir>_test` unit, and Decision 8 only names that spelling.

## Decisions

### 1. A `lyx gate test` shell holds Master's turn end

Master's spawn declares `lyx gate test` beside `masterAwaitedShellPrefix` in `AwaitedShellPrefixes`, so a gate shell outstanding at a Master turn end holds it like a fork, even with both output files present.
Shuttle exports one matcher for awaited labels: a prefix matches at the label's start or at the start of any segment after `;`, `&&`, `||`, `|` or a newline, leading whitespace trimmed.
So `cd <pair> && lyx gate test --tags integration ./x | tail -5` is awaited, whoever launched the shell.
Rationale: the provider reports no pid, so lyx cannot stop a shell; holding the turn end is the only way a gate result is not cut off by strand removal.
Bound: the hold adds only shells with a command-position segment starting with a declared prefix; a prefix inside a quoted argument at a segment's start still matches, which over-holds and never under-holds.
Every held shell, slot-busy re-runs included, counts against `master_timeout_min` and the liveness check, as the recover-batch prefix already does; a hung gate ends the run as a Master timeout, whose strand the next `lyx webster run` reclaims.
An attached or resumed Master already holds every waiting turn end, so the prefix changes nothing there.
`background_shell_wait_min` logging, `Result.EndedShells` and the friction note are unchanged.

### 2. A gate shell outstanding at the run's end is named as discarded

`expiredShellWarning` words a shell that Decision 1's matcher reports as a `lyx gate test` shell as a gate run whose result was never read and does not count as passed.
Rationale: after Decision 1 such a shell outlasts the run only on a timeout or a death, and the outcome must still not read as "tests passed".

### 3. The verify gate's re-prompt names the fixer fork

`GateEntry` gains an optional re-prompt text builder taking the findings path; an entry without one keeps `gateRepromptText`.
Webster's verify-gate entry sets one single-line text: the verify gate refused, findings at the path, do not fix them yourself, spawn the one verify fixer fork with the prompt at `verify-fix.md`'s path, end your turn, and rewrite the outcome and summary on its notification.
The text covers both causes the entry refuses on, a failing verify and a worktree that is not clean.
Master's stencil keys its "A gate failure" section on that text and adds two rules: Master runs no `lyx gate test` and no test itself, and on a gate shell's completion notification it acts on no result and ends its turn, leaving the verdict to the verify gate.
Rationale: today's generic text, "read it, fix every finding, and end your turn", is the instruction Master obeyed in `driver-context`; the rule now arrives where Master reads the failure.
Bound: the change is message and stencil text; attempts, findings files and the run-exit parent-write audit are unchanged.

### 4. A recovery's writes count as the run's own

The recovery spawn records its strand's session id, once `FindRun` returns it, on the batch record in a list carried across a respawn and a re-begin, as `AuditWarnings` is.
A re-entrant attach adds nothing, a record written before the field existed reads as an empty list, `reset --to batch-start` keeps the list, and `reset --to start` archives it with the record.
`loadRunWrites` audits those sessions and returns their writes as a third class beside `Master` and `Forks`.
A recovery session whose transcript is missing yields no writes and is logged at Warn, instead of failing the audit as a missing Master transcript does.
Readers: `writtenWorktreePaths` reads all three classes, so a second recovery's prompt lists the first recovery's uncommitted files under "Written by this run"; the reset's `ownTrackedPaths` adds the recovery class, so a reset restores those paths too; the contract-file evidence (`contractevidence.go`) keeps reading the Master and fork classes only.
Rationale: run-wide attribution by session already exists; the recovery strand was the one session it never audited.
Bound: the missing-transcript leniency applies to the recovery class only; a file in a session with no transcript stays "Not written by this run", the safe side.

### 5. `recover-batch` exits non-zero on a terminal stuck or dead digest

A terminal digest whose status is not `done` returns a failure envelope carrying the same digest fields, `recoveries`, `recovery_retry` and `recovery_failed: true`, with a message naming the way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, else end the run stuck naming the batch.
The `--wait` running snapshot and a done digest still exit 0; a failed batch already exits non-zero with `batch_failed`.
`contracts/specs/refusal-spec.md` gets a webster row for it as a terminal failure report, not a new guard, and Master's failure ladder keys its stuck and dead rungs on `recovery_failed`, not on a retry.
Rationale: a caller reading the exit code sees a failed recovery as a failure.

### 6. The implementer body names its gates as allowed

The body shared by the fork and the recovery prompt states that every card gate and batch gate step, `lyx gate test` with any tags and `lyx loom lint-comments`, is always allowed and is not a `lyx fabric` command.
A card `verify:` step that runs the `tmux` tier runs as `lyx gate test --tags tmux <its packages>`, as the body's raw-`go` section and the settings deny already steer.
A card `verify:` step that runs the `llm` tier is not run by the implementer, and its final reply names it.
Rationale: the fabric audit's spelling rule matches only `lyx fabric`, `weft` and `warp`, so the doubt was the prompt's; `PATTERN-test-speed` has the plan-level verify run no `tmux` tier, so "left to verify" would leave a `tmux` step unrun until Publish.
Bound: the implementer skips only `llm`-tier `verify:` steps; every other step still runs.

### 7. An ambiguous answer over build variants reads as resolved

`internal/planglyph` gets one predicate: an ambiguous answer is a build-variant set when the plan's glyph language (`plan.GlyphLanguage()`) is Go and every candidate sits in one unit, shares one kind and lies in a distinct file, with the files' build constraints pairwise mutually exclusive.
A file's constraint is its `//go:build` line conjoined with the GOOS and GOARCH its file name implies, `_test` suffix included, read from the told worktree with `go/build/constraint`.
Two constraints are exclusive when no assignment of their tags, with at most one GOOS and one GOARCH true and `unix` implied by a Unix GOOS, satisfies both.
A file with a legacy `// +build` line only, a `//go:build ignore`, an unreadable file or an unparseable line reads as not exclusive.
Every planglyph reader of a resolve answer reads a build-variant set as it reads `multipart`: Plan-time resolution raises no `glyph-ambiguous`, `create-not-done` and the new side of `rename-not-done` pass, `answerSymbols` returns every candidate, handle binding binds as over a multipart answer, and `create-already-exists`, `delete-not-done` and the old side of `rename-not-done` still fire, since declarations remain.
Rationale: quarry ignores build constraints, so a member declared once per platform answers ambiguous although it is one member to the toolchain; `StartTime` in `internal/proc` (`proc_linux.go`, `proc_windows.go`) does so today, and `tmuxkit`'s `tmuxAllowed` did in #500 before its workaround.
Bound: only candidates in distinct files of one unit with provably exclusive constraints pass; overlapping, unconstrained or unreadable files keep today's ambiguous verdict.

### 8. `create-not-done` says how the target was looked up

The `create-not-done` and new-side `rename-not-done` details name the resolved key, that it was resolved against the worktree's current tree, the status quarry answered, each candidate's file for an ambiguous answer, and whether the unit was found, reusing `statusFindings`' member-versus-unit wording.
A `not_found` whose package has an external test package adds that members of `package <name>_test` resolve under the `<dir>_test` unit.
No resolver change is made: unexported members and members of same-package `_test.go` files already resolve `found`.
Rationale: at base the detail says only "still does not resolve", so a missing symbol, an ambiguity and a wrong unit read alike.

## Constraints

- `PATTERN-shuttle-provider-seam`: the awaited-shell matcher and the re-prompt builder are provider-neutral and live in `internal/shuttleengine`.
- `PATTERN-stencil-ownership`: the verify-gate re-prompt is a one-line send built around a runtime path, the same class as `gateRepromptText`; the full rule lives in the Master stencil.
- `PATTERN-refusal-way-forward`: Decision 5's envelope names its way forward, lands its spec row and has a test.
- `PATTERN-no-denied-recovery`: no way forward or stencil step names a denied command.
- `PATTERN-test-speed`: Decision 6 routes `tmux`-tier steps through `lyx gate test`, the only route an agent may use.
- `PATTERN-told-geometry`: the build-variant predicate reads files under the worktree root its caller passes.
- `PATTERN-sole-parsers`: build-constraint lines are read with the standard library's `go/build/constraint`.
- `PATTERN-test-economy`: tests extend existing tables at the unit tier where one fits; a card that pushes a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- `PATTERN-documentation-lifecycle`: each package's `doc.go` changes in the same commit as its code.
- Decisions 7 and 8 read quarry v0.2.1's ambiguous candidates (unit, kind, file) and `unit` field; board entry `enclose-kickstart` bumps quarry, so whichever lands second re-checks them.
- Board entry `fork-wait-noise` edits the same stencil sections: this task changes which gates are allowed and the Master rules of Decision 3, that entry owns how a gate is waited on, and whichever lands second merges onto the other's wording.

## Auto-mode assumptions

- Decision 1: hold rather than stop.
- Decision 3: a per-entry re-prompt text rather than a verb that spawns the verify fixer; the "Publish verify attempt 3" in `driver-context` was Webster's verify gate on its third attempt.
- Decision 3: Master runs no gate itself, beyond the board's ask.
- Decision 4: attribute the first recovery's writes rather than restart from a clean tree.
- Decision 5: the observed exit 0 was a stuck or dead digest, since a failed batch already exits non-zero.
- Decision 6: `tmux`-tier steps run through `lyx gate test` rather than being left to verify as the board entry says.
- Decision 7: the build-variant reading lives in lyx, not quarry; a Create target does not exist at Plan time, so it is judged at record-batch, not flagged at Plan time as the board entry asks.
- Decision 8: #511's unexported targets failed as build-variant ambiguity, not as a resolver limit.

## Open risks

- The build-variant predicate needs the known GOOS and GOARCH values, which the standard library does not export; a copy a new Go port outdates reads as not exclusive and keeps today's verdict.
- Exclusive variants that do not cover every platform (Linux and Windows only) pass; the done-check is not a build check.

## Acceptance criteria

- A Master turn end with both output files present and a background `lyx gate test` shell outstanding, spelled bare, behind `cd <dir> &&` or piped to `tail`, holds until the shell's completion; a label that has the prefix only mid-segment does not hold.
- A held gate shell that outlives `master_timeout_min` ends the run as a Master timeout, and the friction note and warning name it as a gate run whose result was never read.
- A gate entry without a re-prompt builder sends today's text; webster's verify-gate refusal, for a failing verify and for a dirty worktree, sends the fixer-fork text on one line.
- A second recovery of a batch whose first recovery left uncommitted writes lists them under "Written by this run"; a path no session wrote stays under "Not written by this run"; a contract file a recovery wrote is not counted as Master's.
- A recovery session id is recorded once per spawn: an attach adds none, a respawn keeps the earlier ids, and a state file without the field loads as an empty list.
- A recovery session whose transcript is missing renders the recovery prompt with its files under "Not written by this run", and a missing Master transcript still fails the audit.
- A reset's own paths include a recovery's tracked writes.
- `recover-batch` exits non-zero on a terminal stuck or dead digest with the digest fields, `recovery_retry` and `recovery_failed` in the envelope, and exits 0 on done and on the `--wait` running snapshot; the refusal-spec row exists.
- The rendered recovery and fork prompts state that card and batch gate steps are allowed, that a `tmux`-tier `verify:` step runs through `lyx gate test --tags tmux`, and that an `llm`-tier step is skipped and named in the final reply.
- A Create of a member declared in two files with exclusive build constraints (tag sets such as `tmux || llm` against `!tmux && !llm`, and `_linux.go` against `_windows.go`) passes `create-not-done` and binds its handle; overlapping constraints, two candidates in one file, candidates of different kinds, a file without a constraint, a `// +build`-only file and an `ignore` file keep the ambiguous refusal.
- Plan-time resolution of an Edit or Delete glyph over a per-GOOS member raises no `glyph-ambiguous`; `delete-not-done` still fires while any variant remains; the later-card reference check over a deleted variant set sees every variant.
- A `create-not-done` detail names the status, the candidates' files for an ambiguous answer, and the `_test` unit hint for a `not_found` in a package with an external test package.

## Notes for the plan writer

- Master's spawn spec and the verify-gate entry are built in `internal/websterengine/runlevel.go`; `gateRepromptText` is in `internal/shuttleengine/gate.go`; `awaitedLabel` is in `wait.go`.
- The recovery prompt's grouping is `uncommittedPathsBlock` in `internal/websterengine/recoverbatch.go`; the CLI's terminal envelope is in `internal/webstercli/recoverbatch.go`.
- `doneCheckVerdicts` in `internal/planglyph/donecheck.go`, `statusFindings` in `resolve.go` and `answerSymbols` in `deleteorder.go` are the readers Decision 7 touches.

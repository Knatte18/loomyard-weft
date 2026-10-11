## Goal

Close the Webster faults outside the audit that stop or mislead runs: a run that ends `done` over an unfinished gate tier, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that leaves its allowed gates unsaid, and Create checks that refuse correct work.
Every claim below is made against the code at base 800a0e892.

## Scope

In scope, one clause per board section, each settled by the Decision of the same number:

1. A gate shell the run starts with a plain `lyx gate test` command line holds Master's turn end; any shell the run still ends over is recorded as discarded.
2. The verify gate's re-prompt sends Master to the fixer fork.
3. A batch's recovery sessions count as the run's own writes; a terminal `stuck` or `dead` recovery digest exits non-zero.
4. The recovery prompt names its always-allowed gates and leaves tmux-tagged steps to verify.
5. The `create-not-done` refusal states how the target was looked up, and the unexported and test-file question is pinned by a test.
6. A member declared once per mutually exclusive build-constraint set resolves for DoneChecks, and the plan validator no longer blocks an existing member of that kind.

Out of scope:

- The second half of #507, a `Create` handle reusing a name the same card deletes.
  The board entry's Expected for #507 covers only the ambiguous per-GOOS answer, so this change does not close #507.
- Changing quarry, a separate module that deliberately ignores build constraints.
- Forks' own gate waits and notice noise (board entry `fork-wait-noise`).

## Decisions

### 1. A gate shell holds the turn end

- **Decision:** `lyx gate test` becomes a second awaited shell prefix on Master's spawn spec, beside `masterAwaitedShellPrefix`.
  A gated Master turn end with all output files present and an awaited shell outstanding is then not counted; the wait continues until the shell leaves the outstanding list, Master wakes on its completion, and its next turn end is the arrival.
  The Master stencil states the matching rule as a constraint on shells Master starts: a gate Master runs itself is its own backgrounded Bash call whose command line starts with `lyx gate test`, and Master reads its result before it writes the contract files.
- **Rationale:** at base the wait already never expires a shell (`background_shell_wait_min` only logs) and `Result.EndedShells` already records every shell outstanding at the end, so the issues' cut-off survives in one path: a gated run with output files present and only unawaited shells outstanding counts at once, and shuttle then removes the strand, killing the shell.
  The awaited prefix is the seam shuttle already offers for this.
- **Bound:** the hold ends at the Master run deadline (`master_timeout_min`) or the liveness check, so a gate shell that never ends ends the run `OutcomeTimeout`, never `done`; a shell that finished but wakes no turn is bounded the same way.
  It skips nothing: output files still decide completion, and every negative finalization still consults `allOutputFilesExist` first (`PATTERN-completion-signal` is untouched).
  A shell whose label does not start with the prefix (a `cd … &&` or `timeout` wrapper) keeps today's path: the run may still end `done` with it killed, and the record below is the only guard.
- **Decision:** lyx stops no background shell, in this change or any other.
  `expiredShellWarning` is reworded so the run warning, the summary section and the friction note say the shell's result was discarded and is part of no verdict.
- **Rationale:** the existing record already surfaces the discard; the wording now states what it means for the outcome.

### 2. The verify gate sends Master to the fixer fork

- **Decision:** `GateEntry` gains an optional re-prompt line builder taking the findings path; shuttle uses it in place of `gateRepromptText` when set, and every other gate keeps the default text.
  Webster's `verify` entry sets it to one line, no newline, that names the findings file, says not to edit anything, and names the Master stencil's section "A gate failure: spawn one fixer fork".
  A test pins that the stencil still carries that heading.
- **Rationale:** `gateRepromptText` tells every gated Master "fix every finding, and end your turn", the latest instruction it reads, which contradicts the stencil; the rule belongs in the message Master acts on.
- **Bound:** wording only; attempts, the findings file and the gate's budget are untouched, and the run-exit parent-write audit remains the backstop for a Master that still edits.

### 3. A batch's recovery sessions are write evidence

- **Decision:** the batch record gains the Claude session ids of its recovery strands, appended at each recovery spawn from the run state's session id and carried across a respawn and a re-begin the way `AuditWarnings` already are.
  `loadRunWrites` audits those sessions too and returns their successful write events in a third field beside Master's and the forks'.
  `writtenWorktreePaths` (the recovery prompt's "Written by this run" grouping) and `ownTrackedPaths` (reset's own-path set) include that field; the contract-file evidence keeps reading Master's and the forks' fields only.
- **Rationale:** `loadRunWrites` audits only Master sessions and their forks, so no recovery edit is ever evidence, and a first recovery's files are foreign to the second by construction.
  Attribution by batch is what the board entry asks for, and it keeps the unfinished work in place.
- **Bound:** only successful `Write`, `Edit` and `NotebookEdit` events from sessions recorded on a batch count; a path changed through Bash stays foreign, the conservative side.
  A recovery session whose transcript is absent counts as no writes and does not fail the spawn; this needs the engine seam to report an absent transcript as a typed condition, with the provider's path knowledge staying in `claudeengine` (`PATTERN-shuttle-provider-seam`), and any other audit error keeps the existing transient refusal.
  Reset's tracked-path allowance widens for every target to paths a recovery wrote, which only exempts them from the refusal on dirty tracked paths outside the run's own writes; a recovery strand is the run's own writer, as a fork is.
- **Decision:** a `recover-batch` call that persists a terminal `stuck` or `dead` digest returns `ok: false` and exits non-zero, with a trailing `way forward:` clause and every digest field (`batch`, `status`, `head_sha`, `dead_reason`, `recovery_retry`, `recoveries`, `warnings`) kept; state persistence and the fabric sync run before the envelope.
  A `done` digest and a `running` snapshot exit 0, and the `batch_failed` refusals are unchanged.
  The Master stencil's ladder rungs for a terminal `stuck` and `dead` recovery are reworded to say the command exits non-zero and Master reads the envelope's `status` and `recovery_retry` as before, with a claims row.
  One refusal-spec row per kind: `stuck` and `dead` with `recovery_retry: false` are correctness halts whose way forward is the ladder's `outcome: stuck`; `dead` with `recovery_retry: true` is transient, with one more backgrounded `recover-batch`.
- **Rationale:** the only terminal non-done paths that exit 0 at base are those two digests, and `noteRefusals` writes a friction note for any `ok:false` envelope, so the dead recovery leaves evidence (`PATTERN-friction-capture`).
- **Bound:** no guard is weakened; the exit code alone changes.

### 4. The recovery prompt names its gates

- **Decision:** `webster-prefix-recovery.md` gains a section stating that the card and batch gate steps, `lyx gate test` and `lyx loom lint-comments` are always allowed and are not fabric commands, that the ban is the shared body's ban on mutating `lyx fabric` verbs, and that a step tagged `tmux` is not run by the recovery but left to the plan verify, and never makes the report `FAILED`.
  A row in `contracts/stencils/claims_test.go` pins the wording.
- **Rationale:** the recovery was unsure which gates the sandbox rule bars and could only compile-check tmux-tagged steps; the prompt is the one place it reads.
- **Bound:** the clause names no command the agents' settings deny (`PATTERN-no-denied-recovery`) and no raw `go` build; every non-tmux gate still runs and still fails the report.
  A batch whose card gates are all tmux-tagged leaves the recovery with nothing to verify until the plan verify.

### 5. The `create-not-done` refusal states its lookup

- **Decision:** the finding's detail names the answer status; for `not_found` it adds the unit's status and the unit searched, and for `ambiguous` the candidates with their files.
  The `not_found` detail with a found unit says the lookup covers the unit's files including `_test.go` files, and that a member of an external test package (`package x_test`) lives in the unit `<dir>_test`.
- **Rationale:** the pinned quarry resolves by declaration with no filter on exported-ness or file kind, so unexported identifiers and same-package test files are not a resolver limit; of the issue's two targets, `tmuxAllowed` is item 6's ambiguity, and `rusageStats` is missing or in an external test unit.
  Today's refusal cannot tell which.
- **Decision:** a DoneChecks test pins the negative: an unexported identifier and an identifier in a same-package `_test.go` file resolve `found` and pass.
- **Bound:** wording and a pin; no verdict changes here.

### 6. Disjoint build constraints are not ambiguity

- **Decision:** one predicate in `internal/planglyph` decides whether an ambiguous answer's candidates carry pairwise disjoint build constraints.
  It requires at least two candidates in at least two different files.
  A file's constraint is its `//go:build` expression conjoined with the GOOS and GOARCH its file name implies.
  Two files are disjoint when no world satisfies both, a world being a GOOS, a GOARCH and a truth value for each other tag the expressions name, with the toolchain's implied tags applied (`android` implies `linux`, `ios` implies `darwin`, `illumos` implies `solaris`, `unix` is the unix GOOS set).
- **Decision:** DoneChecks accepts an ambiguous Create answer whose candidates satisfy the predicate, like `found`.
- **Decision:** the plan validator's resolve status policy turns such an answer on a target that is not a Create group's own from blocking `glyph-ambiguous` into the informational `glyph-build-split`, naming the candidate files.
  The drift repair's post-check shares that policy and follows it.
  Consumers that need the member's file or span, such as the scope guard, read an accepted split answer as its candidates' symbols.
  A Create group's own target that already resolves ambiguous keeps the blocking `create-already-exists`.
  Edit and Delete by glyph on an existing per-GOOS or per-tag member then plan cleanly; `delete-not-done` is unchanged, so a Delete passes only once every declaration is gone.
  This relaxation is licensed by #507, not by the board entry's Expected, which asks only that the validator flag the glyph at Plan time.
- **Rationale:** quarry reports both candidates by design because it does not know the GOOS, GOARCH or tags; the decision belongs in lyx, where tree and constraints are known, and one predicate serves both consumers so they cannot disagree.
- **Bound:** a Create declared in only one arm of a split resolves `found`, so a missing second arm is not detected, as before.
  Nothing else passes: a single candidate, two candidates in one file, a candidate with no constraint, an unreadable file, a constraint that does not parse (legacy `// +build` lines included), an unrecognized file-name suffix or more than eight distinct custom tags leaves the answer ambiguous and blocking.
  A not-yet-declared split Create cannot be flagged at Plan time because nothing in the tree shows it; its first sight is `record-batch`, where it now passes.
  Consumers that read symbols from an answer and treat ambiguous as unreadable (caller coverage, redundant-file-target, delete ordering) skip a split member.

## Constraints

- The Master and recovery stencil edits touch `webster-template-master.md` and `webster-prefix-recovery.md`; `fork-wait-noise` edits the same stencils for forks' gate runs, and Decision 1's prefix match depends only on a command line starting with `lyx gate test`, whether awaited or backgrounded, so the later landing reconciles the Master section.
- `internal/shuttleengine/gate.go` is adjacent to `burler-multi-llm` and `parent-review-hardening`; the added field is additive.
- `proc-child-lifetime` plans Edits of the per-GOOS `runChild` and may need path spelling until Decision 6 lands.
- Refusal text and rows: the changed `create-not-done` detail and the new terminal-digest rows keep a trailing `way forward:` clause, each with a refusal-spec row and a test that reaches it (`PATTERN-refusal-way-forward`); the new text also passes the `PATTERN-no-denied-recovery` scan.
- Stencils stay files under `contracts/stencils/webster/` read at call time (`PATTERN-stencil-ownership`); the re-prompt line is a short Go string like `gateRepromptText`.
- New top-level tests that push a package past its row in `cmd/lyx/testdata/test-budget.yaml` list that file under `Edit:`; tests extend existing tables where one fits (`PATTERN-test-economy`).
- Docs land in the same commit: `doc.go` of `shuttleengine`, `websterengine` and `planglyph`, the refusal-spec rows, and the plan spec's check-ID list.
- No `PATTERN.md` entry changes.

## Auto-mode assumptions

- The three gate-shell issues predate the non-expiring wait and the `EndedShells` record that base already holds, so the counts-at-once path is the remaining fault.
- "recover-batch exited 0 while the batch ended failed and terminal" refers to a terminal `stuck` or `dead` digest, because a `failed` batch already exits non-zero.
- The incident Master received `gateRepromptText`, as every gated Master does.
- The record is written for the base code only.

## Open risks

- A shell a batch fork left running under a `lyx gate test` label now holds Master's final turn end until it exits or the deadline ends the run `OutcomeTimeout`; a hung gate process on Windows is `verify-watchdog`'s gap.
- The prefix matches the start of the label only; a wrapped command falls back to the recorded-discard path and the run may still end `done`.
- A tmux-tagged step is identified by its command naming the `tmux` tag.
- A new GOOS missing from lyx's file-name list reads as unconstrained and stays ambiguous.
- A recovery's Bash-made changes stay foreign; a recovery session id is recorded at spawn only, so a strand relaunched under another session, a crash between spawn and save, and an in-flight record written before this change leave that session's files foreign, the incident's failure for that case.
- If the observed exit-0 case was not a stuck or dead digest, item 3's exit change does not reach it.
- The #507 second half and its issue stay open.

## Acceptance criteria

- A gated Master turn end with all output files present and a shell labeled `lyx gate test …` outstanding is not counted, and the shell's exit followed by a plain turn end finishes the run `done`; a never-ending one ends `OutcomeTimeout` at the Master deadline with the shell in `EndedShells`.
- The same turn end with only a differently labeled shell outstanding counts at once, lists the shell in `EndedShells`, and its run warning says the result was discarded and is part of no verdict.
- The verify gate's re-prompt is one line, contains the findings path and the stencil section's name and no "fix every finding"; a gate without the override sends the default text; the stencil carries the named heading.
- A second recovery's prompt lists files the first recovery wrote under "Written by this run"; a Bash-only change and a path no session wrote list under "Not written by this run".
- A recovery session with no transcript does not fail the spawn and adds no paths; an unreadable transcript keeps the transient refusal.
- Reset's own-path set includes a recovery's successful writes; contract-file evidence is unchanged by them.
- `recover-batch` exits non-zero with `ok: false`, a `way forward:` clause and the digest fields for terminal `stuck` and for terminal `dead` with and without `recovery_retry`, writes a friction note, and has persisted state and synced before returning; it exits 0 for `done` and `running`.
- The rendered recovery prompt contains the always-allowed gates clause and the tmux clause, and the no-denied-recovery scan passes over the new text.
- DoneChecks passes a Create of an unexported identifier and of one declared in a same-package `_test.go` file.
- A `create-not-done` detail names the status, the unit searched for `not_found`, and the candidate files for `ambiguous`.
- DoneChecks passes a Create whose ambiguous candidates are `//go:build tmux || llm` and `//go:build !tmux && !llm`, a `_linux.go` and a `_windows.go` pair, and a `unix`/`windows` pair; it still blocks for a single candidate, two candidates in one file, an unconstrained candidate, overlapping constraints (`_linux.go` beside `_android.go`, `linux` beside `unix`), an unparseable constraint and an unreadable file.
- The plan validator reports `glyph-build-split` as informational for an Edit or Delete target with disjointly constrained candidates, still blocks `glyph-ambiguous` for overlapping ones, and still blocks `create-already-exists` for a Create target that already resolves ambiguous.
- A card editing a split member passes validate, `begin-batch` and `record-batch`, including the scope guard and the drift post-check.

## Notes for the plan writer

- Starting points, non-exhaustive: `internal/shuttleengine` (`gate.go`, `spec.go`, `wait.go`, the engine seam for the typed absent-transcript condition), `internal/websterengine` (`runlevel.go`, `triage.go`, `runwrites.go`, `recoverbatch.go`, `state.go`, `reset.go`, the verify gate wiring), `internal/webstercli/recoverbatch.go`, `internal/planglyph` (`donecheck.go`, `resolve.go`, `create.go`, `drift.go`, the scope guard), `contracts/stencils/webster/`, `contracts/stencils/claims_test.go`.
- `lyx quarry resolve` carries each candidate's file; the predicate needs only those paths and the file contents under the worktree root.
- Check any other reader of `recover-batch` output, such as the sandbox suite, that expects `ok: true` for a stuck or dead digest.
- The shuttle corpus case `master-fork-running` is a real transcript; add no corpus case without a real capture, and test the wait directly.

## Goal

Close six Webster faults that stop or mislead a run outside the audit, as the code stands at base `800a0e892`:

1. A background `lyx gate test` shell still running at Master's turn end is cut off, and the run reads `done`.
2. Master fixes a verify-gate failure itself instead of spawning the one fixer fork.
3. A second recovery of a batch treats the first recovery's uncommitted files as foreign, and `recover-batch` exits 0 on a batch that ended terminal and not done.
4. The recovery prompt does not say which gates it may run.
5. `record-batch` refused Creates of unexported targets with `create-not-done`, and the refusal does not say how the target was looked up.
6. A member declared once per mutually exclusive build-constraint set resolves `ambiguous`, so its Create, Rename or Edit never passes.

## Scope

In:

- `internal/websterengine`: Master's awaited shell prefixes, the verify gate's re-prompt and findings text, recovery write attribution, the recovery prompt's uncommitted-path classing.
- `internal/webstercli`: `recover-batch`'s exit code.
- `internal/shuttleengine`: a per-entry re-prompt instruction on `GateEntry`.
- `internal/planglyph`: the build-variant reading of an ambiguous answer, the reproduction and fix of issue 5, and the done-check refusal detail.
- Stencils: `contracts/stencils/webster/webster-template-master.md` and `contracts/stencils/webster/webster-prefix-recovery.md`.
- Specs and docs that describe the changed behavior: `contracts/specs/refusal-spec.md`, `contracts/specs/webster-spec.md`, and the `doc.go` of each package above.

Out:

- Quarry: no quarry change and no `go.mod` bump; quarry keeps answering `ambiguous` for build-tag duplicates.
- Agent-side readers of raw quarry answers (a fork's `lyx quarry resolve`, review lenses): they keep seeing `ambiguous`; D6 changes only the Go-side policy in `internal/planglyph`.
- The implementer body `webster-body-implementer.md` and fork behavior: the fork's gates and `deviations` paragraph are unchanged; the deviations paragraph and the digest's `deviations` field are `arbiter-fixes`' scope.
- Background shells of producers other than Webster's Master.
- The run-exit audit and its parent-write finding: it stays the backstop for issue 2.

## Decisions

### D1 — A gate shell holds Master's turn end

Master's spawn declares a second awaited shell prefix, `lyx gate test`, beside `masterAwaitedShellPrefix` (`lyx webster recover-batch`).
A background shell whose command starts with it holds Master's waiting turn end like a fork, also when both output files exist, so the verify gate never runs while such a shell is outstanding and the run cannot read `done` over it.
Master's stencil adds one line: a completion notification for a `lyx gate test` shell Master did not start (a fork's leftover) needs no action beyond ending the turn, since the plan-level verify gate decides the run.

Bound: the hold covers only shells whose label starts with `lyx gate test`; a shell with an empty label or a wrapped command (`timeout … lyx gate test`) keeps today's count-at-once behavior.
The hold is bounded by the run's own Timeout and the liveness check, as the recovery prefix already is.
It is per session: a resumed or restarted `lyx webster run` starts a new Master session that does not see the old session's shells and inherits no hold.
Master's run is always gated, because the `verify` entry is always appended to its gate list, even when its attempts are 0.

Rationale: holding makes the outcome true and reuses the existing awaited-prefix mechanism.

### D2 — The verify gate's re-prompt names the fixer fork

`shuttleengine.GateEntry` gains an optional re-prompt instruction: the text that follows `Gate findings recorded at <path> — ` in the one-line re-prompt.
Empty keeps today's text (`read it, fix every finding, and end your turn.`) for every existing entry.
A non-empty instruction that is not one line is refused when the gated spec is validated at start, not at send time.
Webster's `verify` entry sets it to say that Master edits nothing, spawns exactly one fixer fork with the prompt at `verify_fix_prompt_path`, and ends its turn.
`renderVerifyGateFindings` ends every findings text, the dirty-tree case included, with the same instruction; the dirty-tree line stops telling the reader to commit or remove paths itself and says the fixer fork does.
The re-prompt keeps its `Gate findings recorded at` opening, so the Master stencil's section that keys on it stays valid.

Bound: only the webster verify entry sets an instruction; the gate's verdict, attempt count and finalize path are unchanged.

Rationale: shuttle's generic re-prompt tells Master to "fix every finding", which contradicts the stencil's fixer-fork rule at the moment Master reads a failure; the message itself must carry the rule.

### D3 — Recovery strands' writes count as the run's writes

Each recovery spawn's session id is recorded on the batch record, in a list carried across a respawn and a re-begin as `Recoveries` and `AuditWarnings` already are.
The run's write history (`loadRunWrites`) audits those sessions too and returns their writes in a slice of their own beside Master's and the forks', so a second recovery's prompt lists the first recovery's unfinished files under "Written by this run", and the strand keeps them as its starting point after checking them against the card.

Requirement: the id is persisted before anything but a crash inside the spawn can stop the strand, `lyx webster run`'s entry-time stop of a leftover recovery strand included.
If shuttle first knows the id only after the spawn's state save, the id is persisted at that first moment, and a strand stopped before it keeps today's classing.
A recorded session whose transcript cannot be audited (missing, or never written because the strand died at startup) counts as having written nothing, and the prompt or reset that read it carries a warning naming the session; it never fails the verb.
A reset that clears a batch's record (`--to batch-start --batch NN`, `--to start`) and `run --fresh` drop that batch's recorded recovery sessions with it, so the classing never reaches past the attempts the run still stands on.

Bound: only successful writes inside the worktree by sessions a batch record names widen the written set; a path none of them wrote stays foreign.
Contract-file evidence keeps reading Master's writes only.
A reset's own-path set gains the recovery-written paths, so a reset may now discard a recovery's uncommitted files; the discard still goes through fabric's destruction path unchanged.

Rationale: attribution by the batch's recorded sessions keeps the first attempt's work and the one write-history reader.

### D4 — `recover-batch` exits non-zero unless the batch ended done

Every `recover-batch` path that reports a batch terminal and not `done` exits non-zero: a fresh terminal digest whose status is stuck or dead (with or without a recovery left), and a re-entrant call that finds the batch's record already terminal and not done.
The envelope keeps every field its terminal digest carries today, so the digest is read the same way; the error message names the batch, its terminal status and the way forward.
The way forward is the failure-ladder rung the status calls for: `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise end the run stuck naming the batch.
A `refusal-spec.md` row lands with it.
The Master stencil's recover-batch rungs say a non-zero exit that carries `status` is a terminal digest and `status` picks the rung, while a refusal is told by its flags.
A `running` answer after `--wait` elapses and a `done` digest still exit 0.

Rationale: a caller reading the exit code must not see success for a batch that did not finish.
`arbiter-fixes` may change the `deviations` field of the same envelope; D4 adds no field and removes none.

### D5 — The recovery prompt names its gates

`webster-prefix-recovery.md` states:

- `lyx gate test` and `lyx loom lint-comments` are not fabric verbs: the card gates, each card's own `**Verify:**` steps and the batch gate are always allowed, and the sandbox rule's fabric ban never covers them.
- A card `**Verify:**` step whose `--tags` names `tmux` or `llm` is left to verify: the recovery does not run it and names it in its report; the prefix points at `loom-plan-spec.md`'s verify model for which tier runs where, rather than restating it.

Bound: the skip covers only steps tagged `tmux` or `llm`; every other step runs as written.

Rationale: the recovery strand starts cold, so the prompt is its only source for this rule.

### D6 — A build-variant ambiguous answer reads as multipart

`internal/planglyph` decides once, at its batched resolve, whether an `ambiguous` answer is build-variant: every candidate has the same kind and sits in a different file, and no single build configuration selects two of those files.
A configuration is a GOOS, a GOARCH and a set of the custom tags the candidate files name; disjointness is judged by enumerating configurations and matching each file with `go/build`'s own file-matching rules, so the `//go:build` line, `unix` and the `_GOOS`/`_GOARCH` file-name suffixes all count.
The GOOS and GOARCH lists are a fixed table in the code, so the check spawns no process.
A candidate file that cannot be read makes the answer not build-variant, failing closed.

A build-variant answer is turned into a `multipart` one, its candidates becoming its symbols.
The rule for readers: a reader that accepts several declarations of one member passes it (Edit and Uses targets at Plan time, a Create and a Rename's new side in `DoneChecks`, `plan:` handle binding); a reader that asks whether any declaration is still present (Delete, a Rename's old side in `DoneChecks`, the delete-ordering check) or needs exactly one (a Rename's old side at Plan time) keeps reading it as present or refused, as for any `multipart` answer.
A Create target that already resolves build-variant stays the blocking `create-already-exists`, its detail still naming the candidates.
At Plan time a build-variant Edit or Uses glyph is flagged with an informational `glyph-build-variant` finding naming its candidates and files; an ambiguous answer that is not build-variant stays the blocking `glyph-ambiguous`.

Bound: only an answer quarry already marks `ambiguous` can be widened, and only when its candidates share a kind and their files are pairwise disjoint; Delete verdicts never pass on it.

Rationale: a member like `internal/proc#StartTime`, declared in `proc_linux.go` and `proc_windows.go`, is one member to any build, and the toolchain compiles exactly one declaration.

### D7 — A done-check refusal says how the target was looked up

Each `create-not-done` and `rename-not-done` finding's detail names the key resolved (a `plan:` handle's expected glyph), quarry's status, and for `not_found` whether the unit was found, for `ambiguous` each candidate with its file and why the files are not disjoint.
So a member that is missing reads differently from an answer the policy could not accept.
The `card not done` row of `refusal-spec.md` keeps its way forward, `lyx webster recover-batch NN`.

### D8 — Issue 5 is reproduced before it is fixed

The landed code does not show issue 5's cause: at HEAD `tmuxAllowed` is one untagged `var` set by a tagged `init`, and `rusageStats` is one untagged `func`.
The plan reproduces `create-not-done` against a real quarry resolve for: an unexported `plan:` handle for a `func` in a `main` package under `cmd/`, an unexported package-level `var`, a member in a `_test.go` file of the package's own unit, and one in an external `_test` package.
A fault on Loomyard's side (the handle's expected-glyph key, the unit spelling) is fixed in this task.
A quarry-side limit is not fixed here: D7's detail names the status quarry returned so the refusal shows it, and the task's summary names the limit for a quarry task.

## Constraints

- `PATTERN-shuttle-provider-seam`: the gate prefix and the re-prompt instruction are caller data in webster, not provider knowledge in shuttle.
- `PATTERN-stencil-ownership`: the stencil changes are read at call time through `stencilstore`; the one-line re-prompt and the findings text stay Go-rendered, as `gateRepromptText` and `renderVerifyGateFindings` are today.
- `PATTERN-producer-pointer-rule`: the recovery prefix points at the plan spec's verify model, never paraphrasing it.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`: the recovery prefix keeps rendering both markers.
- `PATTERN-glyph-conversion-chokepoint`: D6 reads the file quarry's candidate `Symbol` names; it converts no glyph to a path and parses no glyph string.
- `PATTERN-sole-parsers`: D6 parses Go build constraints, not a plan, discussion, summary or recipe.
- `PATTERN-refusal-way-forward`: D4's non-zero exit and every changed refusal carry a way forward and a `refusal-spec.md` row in the same commit, reached by a test.
- `PATTERN-fabric-destruction-chokepoint`: D3 widens what a reset counts as the run's own; the reset still destroys only through fabric.
- `PATTERN-test-economy`, `PATTERN-test-speed`: scenarios extend existing tables where one fits; D6's disjointness is unit-tested without a Go toolchain run.
- D6 and D7 rely on quarry's ambiguous answer carrying each candidate's file; the v0.3.0 bump `enclose-kickstart` plans keeps it.
- CLAUDE.md "Docs land in the same commit": `websterengine`, `shuttleengine` and `planglyph` `doc.go` change with their code.
- CLAUDE.md "Test budget": a card whose new top-level tests push a package past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.

## Auto-mode assumptions

- "Ended failed and terminal" in issue 3 is read as any terminal status other than `done`.
- Issue 4's "left to verify" is taken as the plan spec states it: verify compiles the tmux tier and Publish runs it; the llm tier is never run by a gate.
- Issue 6's "the plan validator flags such a glyph at Plan time" is met by an informational finding, since D6 lets the glyph pass.

## Open risks

- A gate shell that hangs holds Master until the run's Timeout.
- A GOOS or GOARCH added by a later toolchain is missing from D6's fixed table until the table is updated, so a member split by that port alone stays ambiguous, failing closed.
- `weaver` plans to reuse the recovery prompt and `recover-batch` as its normal path; D5's skip and D4's exit code then apply to that path too.

## Acceptance criteria

- A gated Master run whose turn end leaves a `lyx gate test` shell outstanding with both output files present keeps waiting; it reaches the gate only at a later turn end after the shell ends.
- Any other shell, one with an empty label and a wrapped `lyx gate test` command included, still counts at once as today; a gate shell outstanding at a deadline or liveness end appears in `Result.EndedShells` and the friction note.
- A failing webster verify gate sends a re-prompt that opens `Gate findings recorded at`, forbids Master's own edit and names the fixer fork; a gate entry without an instruction still sends today's text; a multi-line instruction is refused at start.
- The findings text for a dirty tree, a timeout and failing identities each ends with the fixer-fork instruction, and the dirty-tree case no longer tells the reader to commit or remove paths.
- A second recovery's prompt lists a path the first recovery wrote and left uncommitted under "Written by this run", including after `lyx webster run` restarts and stops the first strand.
- A path no recorded session wrote stays under "Not written by this run"; contract-file evidence ignores recovery writes.
- A recorded recovery session with no transcript classes nothing as written and yields a warning; `recover-batch` and reset still succeed.
- After `reset --to batch-start --batch NN` the batch's recorded recovery sessions are gone, and a later recovery classes leftover files as foreign.
- A reset's own-path set includes a path a recorded recovery wrote.
- `recover-batch` exits non-zero with the full digest for a terminal stuck digest, for a terminal dead digest with and without `recovery_retry`, and for a re-entrant call over a terminal non-done record; it exits 0 for `done` and for `running`.
- The Master stencil tells a non-zero terminal digest from a refusal by `status`.
- The recovery prompt names `lyx gate test` and `lyx loom lint-comments` as allowed, and `tmux`- and `llm`-tagged `**Verify:**` steps as left to verify.
- `DoneChecks` passes a Create whose member is declared under `//go:build tmux || llm` and its negation, under `linux` and `!linux`, and in `_linux.go` and `_windows.go` files.
- `DoneChecks` blocks a Create whose candidates overlap (two untagged files, or `linux` beside `unix`) or differ in kind, with a detail naming the status, the candidates, their files and the reason.
- A Delete, a Rename's old side and the delete-ordering check over a build-variant member still read it as present.
- `lyx loom validate-plan` passes an Edit or Uses glyph naming a build-variant member with an informational `glyph-build-variant` finding, and still reports `glyph-ambiguous` for an overlapping one; a `plan:` handle binds to a build-variant member.
- The four D8 cases each either pass `DoneChecks` once created, or fail with a detail naming quarry's status.
- `create-not-done` for a missing member names the resolved key, `not_found` and the unit's status.

## Notes for the plan writer

- Awaited prefix: `masterAwaitedShellPrefix` and `AwaitedShellPrefixes` in `internal/websterengine/runlevel.go`; `awaitedLabel`, `countsAtOnce` and `expiredTurnEnd` in `internal/shuttleengine/wait.go`; the "Background shells in Master's wait" section of `internal/websterengine/doc.go`.
- Re-prompt: `gateRepromptText` in `internal/shuttleengine/gate.go`; `renderVerifyGateFindings` in `internal/websterengine/verifygatereport.go`.
- Recovery attribution: `uncommittedPathsBlock` in `recoverbatch.go`, `loadRunWrites` and `recordedSessions` in `runwrites.go`, `BatchState` in `state.go`; the audit's missing-transcript error is in `claudeengine`'s fork audit.
- Exit code: the terminal `output.Ok` in `internal/webstercli/recoverbatch.go`.
- Done-checks: `doneCheckVerdicts` in `internal/planglyph/donecheck.go`, `statusFindings` in `resolve.go`, `createFindings` in `create.go`; the binding path's status handling is not yet located.

# Decision record: Webster stability — gate shells, Master self-fix, recovery, Create checks

## Goal

Close the Webster faults that stop or mislead runs outside the audit:
a backgrounded gate shell cut off unread when Master's run ends;
Master fixing a verify failure itself instead of spawning the fixer fork;
a second recovery treating the first recovery's uncommitted work as foreign, and `recover-batch` exiting 0 over a batch that ended badly;
a recovery prompt that does not say which gates it may run;
and Create checks that cannot pass, or cannot say why, for a member declared once per build-constraint set or per GOOS.

The record faces the code at base `800a0e892`.

## Scope

In:

1. Master's gated turn end holds on an outstanding `lyx gate test` shell, and a leftover shell is recorded as unread (#505, #509, #517).
2. The verify gate's re-prompt to Master forbids Master's own edit where Master reads the failure.
3. A recovery's uncommitted-path attribution counts the writes of the batch's earlier recoveries (#522); as a deliberate consequence, `lyx webster reset` treats those writes as the run's own.
4. `lyx webster recover-batch` exits non-zero on a terminal `stuck` or `dead` digest (#522).
5. The recovery prompt names the gates it always may run and the tmux- and llm-tagged steps it leaves to verify.
6. `internal/planglyph` treats an ambiguous answer whose candidates are build variants of one member as one member, and the `create-not-done` detail says how the target was looked up (#511, #500, #507).

Out:

- Any change to quarry (`github.com/Knatte18/quarry`), a separate repository; the build-variant rule lives in `internal/planglyph`.
  An external test package's member resolving under `<dir>_test` is quarry's addressing, explained in the detail (Decision 7), not changed.
- The fork implementer prompt (`webster-body-implementer.md`, `webster-prefix-fork.md`): the board scopes item 4 to the recovery prompt, and `fork-wait-noise` is rewriting how forks run gates.
- In-turn gate waits, foreground quick checks and path noise: `fork-wait-noise` owns them.
- #507's second ask, letting a Create handle reuse a name its own card deletes: not in the board's expected outcome, and unneeded once Edit and Delete work by glyph on a per-GOOS member (Decision 6).
- #511's third point, reviving a dead batch through `lyx loom goto`: not in the board's expected outcome.
- `record-batch`'s exit code on a `stuck` fork batch: that digest is the normal entry to the recovery rung.
- A hook or deny that stops Master editing files.

## Decisions

### 1. A `lyx gate test` shell holds Master's turn end

Master's spawn spec declares `lyx gate test` as a second awaited shell prefix beside `masterAwaitedShellPrefix` (`lyx webster recover-batch`).
Shuttle's awaited-shell match (`awaitedLabel`) matches a prefix at the start of the label or at the start of any command in it, splitting the label at `&&`, `||`, `;` and `|` and trimming whitespace, so `cd <pair> && lyx gate test …` and `lyx gate test … 2>&1 | tail -5` both match, whichever signal reported the shell.
The split is a plain token split, not a shell parser.

Rationale: at base, a fresh gated run's first waiting turn end with every output file present counts as an arrival at once when every outstanding task is an unawaited shell (`countsAtOnce`).
Master's gate shells are unawaited, so a Master that wrote its two contract files while one ran finished through the verify gate, and strand removal killed the shell unread.
Holding the turn end lets the shell's completion notification start Master's next turn, whose turn end then arrives at the gate.
The correctness verdict stays the verify gate's, which runs the plan-level verify at every arrival; the hold keeps a gate run Master started from being killed mid-run.
Once `fork-wait-noise` lands and agents run gates in the foreground, this is the backstop for a Master that backgrounds one anyway.

Bound: an awaited gate shell can keep Master's run waiting only until the shell exits, the liveness check, or the run's deadline (`master_timeout_min`).
At the deadline, a run whose output files all exist finalizes `done` (the completion-signal rule, `classifyDeadlineExpiry`), and finalize evaluates the verify gate as at any done; the shell is then ended by strand removal and recorded as unread (Decision 2).
A run missing an output file ends `timeout`.
`lyx gate test` itself is bounded by its slot wait and `go test`'s own timeout.
Only a shell whose label carries an awaited prefix at a command start is held; every other shell keeps today's count-at-once rule.
Only Master's spec gains the prefix, and attached or resumed runs, where nothing counts at once, are unchanged.

### 2. A run's leftover background shells are recorded as unread

`expiredShellWarning`'s text, which is both the run warning and the friction-note line, adds that the run never read the shell's result.

Rationale: after Decision 1 a leftover shell is either not a gate run or one cut off by the deadline; the warning must not leave a reader to infer that its work passed.

### 3. The verify gate's re-prompt forbids Master's own edit

`shuttleengine.GateEntry` gains an optional per-entry re-prompt text, a function of the findings path; an entry without one keeps `gateRepromptText`'s generic line (`Gate findings recorded at <path> — read it, fix every finding, and end your turn.`).
Webster's `verify` entry (`NewVerifyGate`) sets its own line: it opens with `Gate findings recorded at <path>`, the key the Master stencil's "A gate failure" section answers to, then says Master does not edit, fix or commit anything itself and handles the findings as that section says, through a fixer fork, never its own edit.
The line points at the stencil section rather than restating its steps, so the section's own rules, such as `fork-wait-noise`'s rule against a second fixer while one is still running, stay authoritative.
The override's text passes the one-line check every send text passes (`validateSendText`); text that fails it is a wiring error of the entry, never sent.

Rationale: at base the line Master reads at a verify failure tells it to "fix every finding", which contradicts its stencil; in `driver-context` Master followed the line.

Bound: the override changes only the text sent at a failed attempt; attempts, findings files and verdicts are unchanged, and only the verify entry sets one.

### 4. A batch's earlier recoveries are the run's own writers

Each recovery spawn records its shuttle session id, from the run state `recoverSpawn` already reads through `FindRun`, on the batch record, in a list carried to the next fresh record wherever `Recoveries` is carried (a recovery respawn and a re-begin) and cleared wherever `Recoveries` is cleared.
An empty session id records nothing.
Webster's entry-time reclaim (`RemoveRecoveryStrands`) only stops strands and keeps the records, so the list survives a Webster restart between two recoveries.
`loadRunWrites` audits those sessions beside the Master sessions, and their successful writes join the non-Master side of the evidence, never Master's: contract-file evidence keeps judging Master's own writes alone as Master's.
A recorded recovery session whose transcript cannot be read on this machine contributes no writes and is logged at Warn; it never fails the spawn.

Rationale: at base the write evidence covers Master sessions and their forks only, so a recovery strand's writes are attributed to no one and land under "Not written by this run", which the next recovery must leave untouched.
Attribution by the batch's recorded sessions keeps the first attempt's work as the second attempt's starting point, which the recovery prompt already tells it to check against the card.

Bound: the widening adds only sessions the state records for a recovery spawn of this run, so a foreign writer stays foreign.
`loadRunWrites` is also the reset's own-path source, so a reset now accepts a dirty tracked path an earlier recovery wrote instead of refusing on it.
A record without the list, and the window between a strand's start and the list's save, behave as today.

### 5. `recover-batch` exits non-zero on a terminal stuck or dead digest

The call that classifies a recovery terminal `stuck` or `dead` writes an error envelope that still carries every digest field (`batch`, `status`, `head_sha`, `deviations`, `dead_reason`, `elapsed_s`, `recoveries`, `recovery_retry`, `warnings`) and exits non-zero.
Its message names the way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise end the run stuck naming the batch, with the operator's `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN` or `1) lyx webster reset --to start; 2) lyx webster run`.
A `done` digest and a `running` snapshot keep exit 0, and a later call over an already-terminal batch keeps its existing response.
The case lands as a correctness-halt row in `contracts/specs/refusal-spec.md`'s webster section: no later batch may begin over it.
Like every `ok: false` webster envelope it writes a `webster-refusal-recover-batch` friction note, which is wanted, since a failed recovery is reflection evidence.
The Master stencil's recover-batch rungs say the call exits non-zero with the same fields, and read `status` and `recovery_retry` as before.

Rationale: a caller reading the exit code saw success over a batch that ended badly; the kept fields leave Master's ladder unchanged.

### 6. Build-variant members resolve as one member in planglyph

One predicate in `internal/planglyph` classifies an `ambiguous` resolve answer as a build-variant set when every candidate has the same glyph id, each sits in a different file, and each pair of candidate files has mutually exclusive build constraints.
A file's constraint is its `//go:build` line conjoined with its filename's GOOS and GOARCH suffixes, read past a trailing `_test`.
Two constraints are exclusive when no build configuration satisfies both, where a configuration sets at most one GOOS, at most one GOARCH and any set of other tags, with `unix` true exactly for the unix GOOS values.
A candidate with no constraint, an unparseable constraint, an unreadable file, or a configuration the predicate cannot decide is not a build variant: the predicate fails closed to `ambiguous`.

The planglyph readers that ask whether a member exists or resolves use the predicate:

- `DoneChecks`: a build-variant set satisfies `create-not-done` and the new side of `rename-not-done`, and still counts as present for `delete-not-done` and the old side of `rename-not-done`.
- The resolve status policy over Edit and other existing-member targets accepts a build-variant set instead of raising `glyph-ambiguous`, so a plan can Edit or Delete a per-GOOS member by glyph.
- The Create inversion keeps `create-already-exists` for a build-variant set, since the name is occupied.
- `renameDeclSource` keeps requiring `found`, so a handle-shaped Rename of a build-variant member stays refused at Plan time as `rename-old-unresolved`.

Handle binding already counts a build-variant set's declarations as one created glyph, so it needs no change.
A `glyph-ambiguous` finding over candidates that are not build variants names each candidate file and its constraint, so the Plan-time refusal says why.

Rationale: at base DoneChecks accept only `found` or `multipart`, and quarry answers `ambiguous` for a member declared per build-constraint set: `internal/proc#StartTime` over `proc_linux.go`/`proc_windows.go` today, and `tmuxAllowed` in #500's original two-file shape, which the run rewrote into one `var` plus an `init()` to get past the check.
Such a Create can never pass and such a member cannot be edited by glyph.
Unexported members and members in same-package `_test.go` files already resolve `found`, so nothing changes there.
An Edit target ambiguous for any other reason is still refused at Plan time, so the only ambiguity left for record-batch to refuse is a duplicate the implementer created.

Bound: the predicate lets through only candidates that no single build can compile together; two declarations one build could both compile stay `ambiguous` and blocking everywhere.

### 7. `create-not-done` says how the target was looked up

The `create-not-done` and `rename-not-done` details name the resolve key, quarry's status, and the unit searched: the package directory's files including same-package `_test.go` files, with a member of an external test package resolving under `<dir>_test`.
For an `ambiguous` answer they list each candidate file and why the set is not a build variant.
The `card not done` row in `refusal-spec.md` quotes the new detail shape.

Rationale: #511's refusal said only "still does not resolve", which cannot tell a missing symbol from a resolver limit.

### 8. The recovery prompt names its gates

`webster-prefix-recovery.md` gains a short section stating:

- the card gate steps, each card's `**Verify:**` line, the batch gate and `lyx loom lint-comments` are always allowed; the bans cover only `lyx webster` and the mutating `lyx fabric` verbs;
- a step whose tags name `tmux` or `llm`, in any spelling (`--tags tmux`, `--tags=tmux`, `--tags integration,tmux`), is not run by the recovery: it compile-checks the step's own packages with a package-scoped `go vet -tags <tag> <packages>`, never over `...` or `all`, and leaves the run to the plan-level verify and landing's `publish_verify`, per `loom-plan-spec.md`'s verify model;
- such a step never makes the report `FAILED`.

Rationale: the `quick-recipe` recovery was unsure whether gates fell under the fabric ban and could only compile-check tmux-tagged steps; the board asks that those steps be left to verify, and the plan spec gives the tmux tier's run to `publish_verify` and never runs the llm tier in a gate.

## Constraints

- `PATTERN-shuttle-provider-seam`: the awaited-label split reads command text in `shuttleengine`, with no provider payload knowledge; the label still comes from `claudeengine`.
  The split is label matching, shuttle's own decision, not command building, so `PATTERN-shell-mechanics-seam` does not move it into `internal/shell`.
- `PATTERN-completion-signal`: Decision 1 changes only when a waiting turn end counts as an arrival; the deadline's done-when-files-exist answer is untouched.
- `PATTERN-stencil-ownership`: Decision 3's line is one-line send text like the generic re-prompt and `GateResult.Send`, not a prompt file; a websterengine test pins that it opens with the stencil's key.
- `PATTERN-refusal-way-forward`: Decision 5's envelope and Decision 7's detail land with their `refusal-spec.md` rows and tests in the same commit.
- `PATTERN-no-denied-recovery`: no new message or stencil line names a denied form, and Decision 8's vet step is package-scoped so the raw-go deny never refuses it.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`: the recovery and Master stencils keep their directive markers.
- `PATTERN-test-economy`, `PATTERN-test-speed`: new cases are rows in the existing table-driven tests of each surface; planglyph fixtures stay untagged unless they need `go list`.
- Docs in the same commit: `internal/shuttleengine/doc.go` (re-prompt override, awaited match), `internal/websterengine/doc.go` (Master's prefixes, recovery attribution, recover-batch exit), `recover-batch`'s cobra `Long` text in `internal/webstercli`, `internal/planglyph/doc.go` (build-variant rule, check catalogue), `contracts/specs/refusal-spec.md`.

## Auto-mode assumptions

- `lyx gate test` is awaited for Master only, since the board's fault is Master's.
- The board's "exited 0 while the batch ended failed and terminal" is read as a terminal `stuck` or `dead` digest: at base the `batch_failed` path already exits non-zero, and #522's second recovery stopped to ask, which classifies `dead`.
- The board's "the plan validator flags such a glyph at Plan time" is read as: Plan-time checks use the same build-variant predicate as record-batch, so a member ambiguous for any other reason is refused at Plan time with its candidates' constraints named; a Create cannot be judged before it exists.
- Item 4's rule follows the board's "left to verify" for the recovery prompt only.
- The reset accepting an earlier recovery's dirty paths (Decision 4) is accepted as the run's own work, as a fork's is.

## Open risks

- The awaited-label split is a token split: a label quoting `&& lyx gate test` inside a string also matches and holds the turn end, bounded as Decision 1 states.
- A card author's tmux-tagged `**Verify:**` step is run only at landing's `publish_verify` when a recovery implements the card, and not at all where the repo sets no `publish_verify`.
- Forks keep running tmux-tagged `**Verify:**` steps while a recovery compile-checks them; if a fork cannot run them either, the fork prompt needs the same rule, which `fork-wait-noise` or a follow-up decides.
- The GOOS, GOARCH and unix lists the predicate needs are not exported by `go/build`; a stale list fails closed to `ambiguous`, never open.

## Acceptance criteria

1. A fresh gated Master run whose turn end leaves every output file present and only a `lyx gate test` shell outstanding keeps waiting; the shell's completion starts the next turn, and that turn end reaches the verify gate.
2. Labels `cd <dir> && lyx gate test ./x`, `lyx gate test --tags integration ./x 2>&1 | tail -5` and `lyx webster recover-batch 3` hold Master's turn end, both for a payload-reported and a transcript-only shell; an unrelated shell such as `sleep 600` still counts at once.
3. A gated run other than Master's, and an attached or resumed Master run, behave as today.
4. An awaited gate shell still running at the deadline, with both output files present, finalizes `done` through the verify gate, and the shell is warned as unread; with an output file missing the run ends `timeout`.
5. A leftover shell's warning and friction-note line say the run never read its result.
6. A failed verify attempt sends Master a line that opens `Gate findings recorded at`, forbids editing and points at the stencil's gate-failure section; another gated run's failed attempt still sends the generic line.
7. A second recovery of a batch whose first recovery left uncommitted writes lists those paths under "Written by this run", also across a Webster restart and a re-begin between the two; a path no recorded session wrote stays under "Not written by this run".
8. A recovery session id is cleared wherever the batch's `Recoveries` count is cleared; a record without the list renders as today.
9. A recorded recovery session with no readable transcript logs and leaves its paths foreign, without failing the spawn.
10. A recovery strand's write to Master's contract file is never counted as Master's write by the contract-file evidence.
11. `reset` treats a dirty tracked path an earlier recovery wrote as the run's own.
12. `recover-batch` exits non-zero with the full digest fields on a terminal `stuck` or `dead`, naming the way forward that matches `recovery_retry`, and writes its refusal friction note; `done` and `running` exit 0.
13. The recovery prompt lists the allowed gates and the tmux and llm rule, including the `=` and comma-list tag spellings; the stencil claims test pins them.
14. A Create of a member declared in two files under `//go:build tmux || llm` and `//go:build !tmux && !llm` passes `create-not-done`; so do members declared in `x_linux.go` and `x_windows.go`, in `x_linux.go`, `x_darwin.go` and `x_other.go` (`//go:build !linux && !darwin`), and in `x_linux_test.go` and `x_windows_test.go`.
15. Candidates under `//go:build linux` and `//go:build linux && amd64`, or with one candidate unconstrained, stay `ambiguous` and block, and the finding names each file's constraint.
16. A Delete of a build-variant member with one variant left blocks `delete-not-done`.
17. An Edit target on a per-GOOS member passes the Plan-time status policy; a Create handle naming an existing build-variant member raises `create-already-exists`.
18. A Create handle implemented as a build-variant pair binds without `bind-count-mismatch` (a regression pin of today's binding).
19. A `create-not-done` detail names the resolve key, the status and the unit searched, and points at `<dir>_test` for an external test package.

## Notes for the plan writer

- Shell matching: `internal/shuttleengine/wait.go` (`awaitedLabel`, `countsAtOnce`); Master's spec in `internal/websterengine/runlevel.go`.
- Re-prompt: `internal/shuttleengine/gate.go` (`gateRepromptText`, `GateEntry`); the verify entry in `internal/websterengine/verifygate.go`.
- Recovery attribution: `internal/websterengine/runwrites.go`, `recoverbatch.go` (`uncommittedPathsBlock`, `recoverSpawn`), `beginbatch.go` (the re-begin carry), `state.go` (`BatchState`), `contractevidence.go`, `reset.go`.
- recover-batch CLI: `internal/webstercli/recoverbatch.go`, `refusalnote.go`; Master rungs in `contracts/stencils/webster/webster-template-master.md`.
- Recovery prompt: `contracts/stencils/webster/webster-prefix-recovery.md`, claims in `contracts/stencils/claims_test.go`.
- Glyph checks: `internal/planglyph/donecheck.go`, the resolve status policy (`resolve.go`), `create.go`, `handle.go`; `go/build/constraint` parses `//go:build` lines.
- Overlapping board work, textual merges only: `fork-wait-noise` edits the Master template's gate-failure section; `shuttle-startup-ready` and `parent-review-wait` edit `internal/shuttleengine/wait.go`, and `parent-review-wait` also `gate.go`; `plan-glyph-tuning` adds a per-handle bind trace line at record-batch, which should report a build-variant bind as one bound handle.

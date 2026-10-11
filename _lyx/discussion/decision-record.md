# Decision record — dbench-ohsm

## Goal

Close the Webster faults outside the audit that stop or mislead runs:
a background gate shell cut off at Master's turn end, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, `recover-batch` exiting 0 on a terminal batch, a recovery prompt that does not name its allowed gates, and Create/Edit/Delete checks that can never pass for a member declared once per build-constraint set.

## Scope

In:

1. Gate shells: a running `lyx gate test` shell holds Master's turn end (#505, #509, #517).
2. Master self-fix: the verify gate's re-prompt names the fixer-fork route instead of shuttle's generic "fix every finding" text, and Master's stencil states the rule where it reads the failure.
3. Recovery attribution: an earlier recovery's writes count as the run's own in a later recovery's prompt (#522); `recover-batch` exits non-zero when the batch ends terminal.
4. Recovery prompt: names the card and batch gates as allowed, and leaves `tmux`-tagged runs to Publish.
5. `create-not-done` refusal: says how the target was looked up (#511).
6. Build-constraint members: an ambiguous answer whose candidates sit in Go files with pairwise disjoint build constraints counts as one member, at plan time and at record-batch (#500, #507 first remedy).

Out:

- Changing quarry itself; it is the external module `github.com/Knatte18/quarry`, and lyx reads its answers.
- #507's second remedy: a `Create` handle reusing a name every declaration of which the same card deletes.
- #511's third point: reviving a dead batch through `lyx loom goto`.
- Restoring the tree to clean before a recovery.
- Giving fork implementers the recovery prompt's gate-allowance wording.

## Decisions

### D1 — A running `lyx gate test` shell holds Master's turn end

Decision: Master's spawn declares a second awaited shell prefix, `lyx gate test`, as a constant beside `masterAwaitedShellPrefix` in `internal/websterengine`.
A shell matching it holds a gated turn end with every output file present, as a fork does, until the shell exits or the run's `Timeout` ends the run.
When the shell exits, Master gets its completion notification, and its next turn end with nothing outstanding arrives at the gate.
`background_shell_wait_min` stays a logging and wait-marker threshold and expires nothing.
On a deadline or died outcome the existing `EndedShells` record, `webster-background-shell` friction note and `RunResult.Warnings` entry report the shell; their wording states that the shell's result was never read, so the outcome cannot read as tests passed.

Bound: the hold covers only shells whose command matches an awaited prefix by D2's rule; every other shell keeps today's turn-end rule.
A hung gate shell holds the run at most until `Spec.Timeout`, the bound an awaited `recover-batch` shell already lives under.

Rationale: lyx cannot stop a shell (the provider reports no pid), so holding is the only route that never reports `done` with a tier unfinished; ending the run stuck on every outstanding gate shell would stop runs that are fine.

### D2 — Awaited prefixes match at the start of any command segment

Decision: shuttle's `awaitedLabel` matches a prefix at the start of the label or at the start of any segment after `&&`, `||`, `;` or `|` outside quotes, after skipping a leading `cd <dir>`, environment assignments and `(`.
A label it cannot split falls back to plain `strings.HasPrefix`.
The matching stays caller-data-driven and provider-neutral in `internal/shuttleengine`; `Spec.validate` still does not inspect the prefixes.

Bound: a quoted occurrence (`echo "lyx gate test"`) never matches, and the widening applies to both of Master's prefixes and to any other caller's.

Rationale: a shell's `Label` is its full command (the payload's or transcript's Bash `command`, its description only as a fallback), and agents write both `lyx gate test … 2>&1 | tail` and `cd d && lyx gate test …`.

### D3 — A gate entry may carry its own re-prompt text

Decision: a `GateSpec` entry gains optional caller-supplied re-prompt text, a function of the findings path returning one line;
`gateRepromptText`'s generic wording stays the fallback when an entry sets none.
Webster's verify-gate entry sets a line telling Master not to edit any file and to spawn the one verify fixer fork per its stencil's gate-failure section, naming the path of the verify fixer prompt file Go renders at run entry.

Bound: the text is one line (`validateSendText` refuses a newline), carries no provider knowledge, and changes only what is sent, never whether or when the gate re-prompts.

Rationale: the generic re-prompt reads "read it, fix every finding, and end your turn", which instructs Master to fix the failure itself against its stencil; the message Master acts on must carry the rule.

### D4 — Master's stencil states the rule where it reads a verify failure

Decision: `webster-template-master.md`'s gate-failure section opens with the rule, in one place: Master never edits, stages or commits a file on a gate failure, and the fixer fork is the only route.
The section points at `webster-body-verify-fix.md` and does not restate it (`PATTERN-producer-pointer-rule`).

Rationale: the board asks for the rule where Master reads the failure, not only in its rules list; D3 makes that the re-prompt, and the stencil section is the other place Master reads.

### D5 — Recovery sessions are audited as the run's own writes

Decision: each recovery spawn's Claude session id is recorded on the batch's record in `state.json` as an append-only list that survives a respawn and a re-begin.
It is captured at spawn; where the shuttle API yields the id only later, the next recovery recovers it from the recorded `ShuttleRunDir` before rendering.
`loadRunWrites` audits those sessions too and returns their writes in a recovery group of `RunWrites`, never folded into `Master`.
Every consumer of `RunWrites` treats the recovery group as the run's own:
the recovery prompt classes those paths "Written by this run";
reset's own-path set includes them;
contract-file evidence treats a recovery's write to a contract file as it treats a fork's.

Bound: only sessions a recovery spawn of this run recorded are audited; a write by any other session stays "Not written by this run".
A session whose transcript cannot be found is an audit error naming the session, as today.

Rationale: a recovery strand is its own Claude session and is never audited today, so its writes always class as foreign; attributing by recorded session keyed to the batch also fits a future non-fork implementer (`weaver`, `webster-parallel-execution`), which records its sessions the same way.

### D6 — `recover-batch` exits non-zero when the batch ends terminal

Decision: a terminal digest of `status: stuck`, or `status: dead` with `recovery_retry: false`, returns through `output.ErrFields` carrying the same digest fields (`status`, `dead_reason`, `recovery_retry`, `recoveries`, `warnings`), so the call exits non-zero.
`done` and `dead` with `recovery_retry: true` stay `output.Ok`.
The error message names the way forward: Master writes `outcome: stuck`, and the operator replans or resets the batch.
The refusal gets its row in `contracts/specs/refusal-spec.md`'s webster section.
Master's stencil rungs for those two outcomes say the call ends non-zero with that digest, and stay keyed on `status`.

Rationale: an exit 0 on a terminal batch misreads as success to any reader of the exit code; keeping the digest fields keeps Master's existing rungs matching with no second signal.

### D7 — The recovery prompt names its allowed gates

Decision: `webster-prefix-recovery.md` states, once, that every step under the card gates and the batch gate, `lyx gate test` and `lyx loom lint-comments` included, is always allowed and is not a fabric verb.
It states that a step whose `--tags` names `tmux` is compiled with `go vet -tags tmux <packages>` and not run, leaving the run to Publish's `publish_verify` tier, and the card counts as passed on its other steps.
The wording gives no reason for leaving the tier to Publish.
The shared `webster-body-implementer.md` is unchanged.

Bound: the allowance covers only the steps the rendered gate lists hold; the sandbox rule's ban on mutating `lyx fabric` verbs stands, and no named command is one the agents' settings deny (`PATTERN-no-denied-recovery`; the raw-go deny covers `go test` with a `tmux` tag, not `go vet`).

Rationale: the recovery strand could not tell whether gates fell under the fabric ban; the plan format already leaves the `tmux` tier to Publish, so the recovery matches the plan's own `## verify:`.

### D8 — Disjoint build constraints make an ambiguous answer one member

Decision: `internal/planglyph` gains one predicate over an ambiguous answer's candidates: true when every candidate is a Go file and every pair of candidate files has build constraints that cannot both hold.
A file's constraint is its `//go:build` line, read with `go/build/constraint`, conjoined with its filename's `_GOOS`/`_GOARCH` suffix.
The test enumerates assignments over the atoms the two constraints use, with known implications applied (`unix` covers the Unix GOOS set, `android` implies `linux`, `ios` implies `darwin`, at most one GOOS and one GOARCH true).
The predicate is conservative: a non-Go candidate, a file with no constraint, an unreadable or unparsable file, or more atoms than the enumeration cap answers not disjoint.

Every reader of an ambiguous answer applies it:

- the `create-not-done` done-check and a Rename's new side count a disjoint ambiguous answer as resolved;
- the plan-time status policy accepts it for `Edit:`, `Uses:` and `Delete:` targets, raising no `glyph-ambiguous`;
- `delete-not-done`, a Rename's old side and `create-already-exists` keep treating it as present;
- a non-disjoint ambiguous answer keeps raising `glyph-ambiguous` at plan time, its detail naming each candidate with its constraint.

Every consumer downstream of a plan-time pass (`DetectDrift`, handle binding, scope guard, delete order) tolerates one glyph naming several declarations for a disjoint ambiguous answer, or treats it as it treats a multipart answer.

Bound: the predicate only reclassifies quarry's `ambiguous`; it never turns `not_found` or a rejection into resolved, and never applies to a non-Go language.

Rationale: quarry ignores build constraints, so an idiomatic per-GOOS or per-tag member can never pass today; quarry is an external module, and its candidates carry `File`, so lyx can decide this itself.

### D9 — `create-not-done` says how the target was looked up

Decision: the `create-not-done` detail names the resolved key (a `plan:` handle's expected-glyph half), quarry's status, and for an ambiguous answer the candidate files with their constraints and why they are not disjoint.
Rename-new's and the plan-time `glyph-ambiguous` details follow the same shape.
The `DoneChecks` doc comment and `planglyph`'s `doc.go`, which say a Create resolves only found or multipart, change in the same commit.

Rationale: #511's targets were per-build-constraint members (unexported and `_test.go`-only members resolve `found`), and the old detail ("still does not resolve") could not tell a missing symbol from a resolver limit.

## Constraints

- `PATTERN-refusal-way-forward`: D6's refusal and every reworded webster/loom refusal (D9) keep a way forward, a row in `contracts/specs/refusal-spec.md` and a reaching test, in the same commit.
- `PATTERN-no-denied-recovery`: D3's re-prompt and D7's prompt name only allowed commands.
- `PATTERN-shuttle-provider-seam`: D2 and D3 stay provider-neutral in `internal/shuttleengine`; provider label extraction stays in `claudeengine`.
- `PATTERN-completion-signal`: D1 changes when a turn end counts, not how a negative finish answer consults `allOutputFilesExist`.
- `PATTERN-shuttle-stop`: no code stops a shell or strand outside shuttle's stop verb; D1 adds no stop.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`, `PATTERN-parent-directive`, `PATTERN-edit-directive`: D4 and D7 edit stencils read from the told directory, point instead of paraphrase, and keep the rendered directives.
- `PATTERN-sole-parsers`: D8 adds no parser of plan files; it reads Go source headers through `go/build/constraint`.
- `PATTERN-friction-capture`: the `webster-background-shell` note keeps being written on every outcome with a shell.
- `PATTERN-test-economy`, `PATTERN-test-speed`: extend existing tests (`donecheck_test.go`, `resolve_test.go`, `recoverbatch_git_test.go`, `verbs_test.go`, shuttle wait tests, `contracts/stencils` claims tests) before adding files; a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- CLAUDE.md docs rule: `doc.go` of `shuttleengine`, `websterengine` and `planglyph`, and the webster spec where it states the recover-batch exit, change in the same commit as the code.

## Auto-mode assumptions

- The session ran autonomously; every decision is a chair pick weighed against advisor `ly:dbench-ohsm:discussion-advisor-1`, which agreed with each.
- D1's hold, not an explicit stop, is the board's first option; lyx has no pid to stop a shell with.
- D7 reads the board's "left to verify" as Publish's `publish_verify`, since the plan's `## verify:` compiles the `tmux` tier and never runs it.
- #511's premise (unexported targets fail) is taken as not reproducing: on this tree unexported and `_test.go`-only members resolve `found`, and both cited targets were per-build-constraint members at the time.
- #507's second remedy and #511's dead-batch revival are left out, since the board entry's expected outcomes do not name them.

## Open risks

- D8: a disjoint ambiguous Edit/Delete target names several declarations under one glyph; a consumer that assumes one symbol per found glyph (kick-start spans, handle binding's `renameSignature`, delete order's `answerSymbols`) may mishandle it. The plan must check each and treat the answer as multipart where that is correct.
- D8: GOOS/GOARCH and `unix` knowledge is a fixed table in lyx; a Go release adding a port can make the predicate answer not disjoint until updated, which fails closed.
- D5: if a restart kills a recovery strand before its session id is recorded, and its run directory is gone, its writes stay foreign.
- D1: a hung gate shell now holds Master until the run's `Timeout`.
- D2: if a provider ever truncates the payload `command`, a mid-command match can miss and the shell falls back to today's rule.

## Acceptance criteria

- A gated Master turn end with every output file present and an outstanding shell whose command is `lyx gate test …` (directly, after `cd d &&`, or piped into `tail`) is held, not counted as an arrival; the shell's exit followed by a quiet turn end arrives at the gate.
- A quoted `"lyx gate test"` inside another command, and an unrelated shell, keep today's turn-end rule.
- A run ending on its deadline with a gate shell outstanding reports it in `EndedShells`, the friction note and `RunResult.Warnings`, with wording that its result was never read.
- A verify-gate failure re-prompts Master with Webster's own one-line text naming the fixer fork route; a gate entry with no text still sends the generic line.
- A second recovery's prompt lists the first recovery's uncommitted writes under "Written by this run"; a path no recorded session wrote stays under "Not written by this run".
- Reset's own-path set and contract-file evidence include a recovery's writes.
- `recover-batch` exits non-zero with the digest fields on `stuck` and on `dead` without retry, and exits 0 on `done` and on `dead` with retry; the refusal has its spec row.
- The rendered recovery prompt names the gates as allowed and leaves `tmux`-tagged steps to Publish, and the stencil claims tests stay in step.
- `create-not-done` passes for a member declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`), and for a member in `x_linux.go` and `x_windows.go`.
- A plan `Edit:` or `Delete:` of a per-GOOS member validates with no `glyph-ambiguous`; `delete-not-done` blocks while any of its declarations remains.
- Two candidates with overlapping constraints, a candidate with no constraint, or a test and non-test file with the same constraint stay ambiguous at plan time and at record-batch, the detail naming each candidate's constraint.
- A `create-not-done` detail names the looked-up key and quarry's status.

## Notes for the plan writer

- Item 1: `internal/shuttleengine/wait.go` (`awaitedLabel`, `expiredTurnEnd`), `internal/websterengine/runlevel.go` (`masterAwaitedShellPrefix`, the `AwaitedShellPrefixes` spawn field), the shell note wording near `internal/websterengine/doc.go`'s background-shell section.
- Item 2: `internal/shuttleengine/gate.go` `gateRepromptText`; the Master stencil's "A gate failure: spawn one fixer fork" section.
- Item 3: `internal/websterengine/runwrites.go` (`loadRunWrites`, `recordedSessions`), `recoverbatch.go` (`uncommittedPathsBlock`), `reset.go`, `contractevidence.go`, `state.go` (`BatchState`); the exit in `internal/webstercli/recoverbatch.go`.
- Item 6: `internal/planglyph` `donecheck.go`, `resolve.go`, `create.go`, `drift.go`, `deleteorder.go`, `handle.go`.

## Goal

Close the Webster faults that stop or mislead runs outside the audit (board entry `twb-twp-2`, issues #505, #509, #517, #522, #511, #500, #507):
a gate shell Master started is cut off when its run finishes,
Master fixes a verify failure itself because the gate's re-prompt tells it to,
a second recovery treats the first recovery's work as foreign and `recover-batch` exits 0 on a failed batch,
the recovery prompt leaves its allowed gates unclear,
and a Create or Edit of a member declared once per build-constraint set can never pass.

The design is written against base `800a0e892`.

## Scope

In:

1. Shuttle's awaited-shell match and Master's awaited set, so a running `lyx gate test` shell holds Master's turn end.
2. A caller-supplied re-prompt text on a shuttle gate entry, used by Webster's `verify` entry.
3. Recovery-strand sessions as run write evidence, and a non-zero `recover-batch` exit on a terminal digest that is not done.
4. The implementer body's gate wording (shared by forks and the recovery strand).
5. The text of the done-check findings, naming how the target was looked up.
6. Build-variant members in `internal/planglyph`: an ambiguous Go answer whose candidates are mutually exclusive build variants reads as one member, at Plan time and in the done-checks.

Out:

- Reviving a dead batch through `lyx loom goto` (#511's third item); the board entry does not ask for it.
- Letting a Create handle reuse a name its own card deletes (#507's first option); item 6 makes Edit and Delete of a per-GOOS member work, which #507 accepts as the alternative.
- Renaming a build-variant member by Rename pair.
- Changing quarry (`github.com/Knatte18/quarry`, a pinned external module).
- How forks run gates and quick checks (background versus foreground, `cd`), which is `fork-wait-noise`'s.
- Stopping a non-gate shell lyx cannot see a pid for.

## Decisions

### D1. A running gate shell holds Master's turn end

Master's spawn declares `lyx gate test` as a second awaited shell prefix beside `lyx webster recover-batch`.
Shuttle's awaited match changes from "the label starts with a prefix" to "some command segment of the label starts with a prefix": the label is split on `&&`, `||`, `;`, `|` and newlines, each segment trimmed of leading whitespace.
So `cd <pair> && lyx gate test --tags integration ./x | tail -5` is awaited.
The Master template says what Master does on a gate shell's completion notification: it reads nothing into it, edits nothing, and ends its turn, since the verify gate verifies at the arrival that follows.

Rationale: the label is the shell's command text (payload or transcript), and Master's gate shells in #505/#509/#517 were wrapped or piped, which a plain prefix misses.
At base, a gated run's waiting turn end with every output file present finishes Done when every outstanding task is an unawaited shell (`expiredTurnEnd`), so the run ended `done` and strand removal killed the shell.
An awaited gate shell holds that turn end until the shell's notification gives Master a new turn, whose end arrives at the gate with nothing outstanding.

Bound: the match only adds holds; it lets nothing through that base held.
A held gate shell is bounded by Master's run `Timeout` and the liveness check, as every awaited task is, and a run ending on its deadline records the shell on `Result.EndedShells` as at base.
The split over-matches quoted text such as `echo "a; lyx gate test"`, which only holds that shell until it exits, and under-matches wrappers such as `bash -c "…"` or `timeout 900 lyx gate test`, which keep base behaviour.

### D2. The verify gate's re-prompt sends Master to the fixer fork

`shuttleengine.GateEntry` gains an optional re-prompt text, a function of the findings path; nil keeps today's `gateRepromptText`.
Webster's `verify` entry, built in `runlevel.go` beside `verifyGateName`, supplies one that names the findings file, says Master does not edit, stage or commit any file, points at its template's "A gate failure: spawn one fixer fork" section and the rendered fixer prompt path, then says to end the turn.
Every path that builds Merriam's gate spec, a fresh start, a resume and an attach, carries the same entry.
The text points at the template section and does not restate the fork prompt (PATTERN-producer-pointer-rule), and names no denied command (PATTERN-no-denied-recovery scans it as a webster string literal).

Rationale: at base the only text Master reads at a verify failure is shuttle's generic "read it, fix every finding, and end your turn", which contradicts the template's "Do NOT localize or fix the failure yourself", and Master followed the message it was handed.

Bound: the text changes what is typed and nothing about the gate's verdict, attempt count or the run-exit audit, which still catches a parent write.

### D3. Recovery strands' writes count as the run's own

Each recovery spawn that starts records the strand's session id on its batch record, read from the started run's record (`run.json`'s session id, or an accessor on the started run).
The list is carried across re-begin and respawn the way `Recoveries` and `AuditWarnings` are carried, so a respawn's fresh record keeps every earlier recovery's session.
A start that fails records nothing, and an attach that does not spawn adds nothing.
`RunWrites` gains a recovery-session field, filled by auditing each recorded recovery session's transcript;
a recorded session whose transcript does not exist contributes no writes and adds a logged warning, and never fails the read.
The recovery prompt's uncommitted-paths block and a reset's own-path set read the field;
the contract-file evidence keeps reading Master's and forks' writes and never the recovery field.

Rationale: at base `loadRunWrites` audits only Master sessions and their forks (`recordedSessions`), a recovery batch's `SessionID` is empty, and a respawn's fresh `BatchState` replaces the prior `ShuttleRunDir`, so the first recovery's files fell under "Not written by this run".

Bound: only transcripts of strands the state records as this run's recoveries count; a path no recorded session wrote stays "Not written by this run".
Attribution is run-wide, the same scope base applies to Master's and forks' writes, rather than per batch as the board words it:
a later batch begins only after earlier batches are terminal done, so another batch's uncommitted recovery work cannot sit in the tree when a batch is recovered, and splitting fork writes per batch is not asked for.
The reset reads the same evidence, so a reset proceeds over dirty paths a recovery wrote, which are the run's own work by the same rule as a fork's.

### D4. `recover-batch` exits non-zero on a terminal digest that is not done

Every terminal digest whose status is not `done` makes `recover-batch` exit non-zero.
Its envelope is an error envelope naming the batch and its status, carrying the same digest fields, `recoveries` and `recovery_retry` as today's success envelope.
Its way forward follows the existing rungs: `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise end the run stuck naming the batch.
A `done` digest and the `--wait` running snapshot keep exit 0.
The Master template states that its "completes with `status: …`" rungs, the `dead` with `recovery_retry: true` retry included, read those fields from the non-zero envelope, and that it is not a refusal.
The refusal is a row in `contracts/specs/refusal-spec.md`'s webster section, per PATTERN-refusal-way-forward.
The plan confirms no caller other than Master's template reads the exit code.

Rationale: at base every terminal digest goes through `output.Ok`, so a batch that ended failed and terminal reported exit 0 (#522); `batch_failed` already exits non-zero.

### D5. The implementer body names its always-allowed gates

The implementer body's sandbox rule, shared by forks and the recovery strand, gains two statements and changes no other wording (`fork-wait-noise` owns how gates are run):

- `lyx gate test` and `lyx loom lint-comments` are not `lyx fabric` verbs: the card gate, the batch gate and a card's own `**Verify:**` steps are always allowed.
- A `**Verify:**` step tagged `tmux` or `llm` is not run by the strand and is no failure: the strand names it in its report, and the tier is compiled by the plan's `## verify:` and run at Publish, per the plan spec's tmux-tier rule, which the body points at.

The body's "A module-wide run or a `tmux`-tier run goes through `lyx gate test`" becomes "A module-wide run goes through `lyx gate test`".
`claudeengine`'s raw-go deny steer is unchanged.

Rationale: the recovery strand in #522's run could not tell the gate commands from the fabric ban and could only compile-check its tmux-tagged steps; the rule lives in the body both roles read, so one statement covers both.

Bound: the tmux and llm skip lets a card's tagged `**Verify:**` step pass the batch unrun; the plan-level verify compiles it and Publish runs it, as for every other tmux-tier test.

### D6. Build-variant members read as one member

In `internal/planglyph`, an ambiguous answer is a build-variant set when the plan's glyph language is Go and every candidate has the same id and kind, in distinct files of one package, whose build constraints are pairwise disjoint.
A file's constraint is its `//go:build` expression conjoined with its `_GOOS`, `_GOARCH` or `_GOOS_GOARCH` file-name suffix, read with `go/build/constraint`.
Two constraints are disjoint when no tag assignment satisfies both, where at most one GOOS and one GOARCH value is true and `unix` is true exactly for the unix GOOS values.
The GOOS, GOARCH and unix lists are lyx's own closed list, since `go/build` does not export the toolchain's; a value missing from it reads as an ordinary tag.
A file whose constraint cannot be read, or any overlapping pair, keeps the answer plainly ambiguous.

One helper decides the variant reading, and the readers that change are:

- Plan-time status findings for Edit, Delete and `Uses:`: pass like `multipart`, so a per-GOOS member can be edited or deleted by glyph.
- `createFindings`: `create-already-exists`, as for any existing member.
- `DoneChecks`: a Create target and a Rename's new side count as resolved; Delete and a Rename's old side still require `not_found`.
- Delete ordering (`deleteorder.go`) and the scope guard: every candidate's file and span is the member's.

A Rename pair whose old side is a variant set stays refused (`renameDeclSource` derives from exactly one declaration).
A Plan-time ambiguous answer that is not a variant set keeps `glyph-ambiguous`, and its detail says which candidate files overlap or which constraint could not be read.

Rationale: quarry ignores build constraints, so `tmuxAllowed` in `allow_tmux.go`/`allow_none.go` and per-GOOS `StartTime` or `runChild` answer `ambiguous`, which no Create done-check or Edit glyph accepts (#500, #507).
Disjoint constraints mean no build sees two declarations, which is Go's own one-member rule.

Bound: the reading relaxes only an ambiguous Go answer, and only when every candidate pair is provably disjoint; anything else fails closed as today.

### D7. Done-check findings say how the target was looked up

Every `create-not-done`, `delete-not-done` and `rename-not-done` detail names the resolved key and quarry's status against the worktree tree, and:
for `not_found`, whether the unit was found;
for `ambiguous`, each candidate file and why it is not a variant set (D6).

Rationale: quarry resolves unexported and `_test.go` members (`cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed` both answer `found` today, each a single declaration), so #511's failures were not an unexported-name limit.
At base the `create-not-done` and new-side `rename-not-done` details name no status, so a missing symbol cannot be told from a resolver answer.

## Constraints

- PATTERN-shuttle-provider-seam: the segment match reads the provider-neutral `BackgroundTask.Label`; nothing under `claudeengine` changes for D1.
- PATTERN-shell-mechanics-seam: D1 parses a label and builds no pane-shell command string.
- PATTERN-refusal-way-forward and PATTERN-no-denied-recovery: D4's error envelope lands its spec row and a test; D2's text and D7's details name no denied command.
- PATTERN-producer-pointer-rule: D2's re-prompt and D5's body text point at the template section and the plan spec rather than restating them.
- PATTERN-sole-parsers: Go build constraints are read with `go/build/constraint`; no plan, discussion, summary or recipe parse moves.
- PATTERN-test-economy and PATTERN-test-speed: extend the existing tables (`awaitedLabel`, `expiredTurnEnd`, `uncommittedPathsBlock`, `doneCheckVerdicts`, `statusFindings`, the template claims) before adding tests; the constraint logic is a pure function tested without git or quarry.
- Docs in the same commit: each touched package's `doc.go`, the `recover-batch` Long help, the Master template and the implementer body.
- `cmd/lyx/testdata/test-budget.yaml` goes under a card's `Edit:` if new top-level tests pass a package's row.

## Auto-mode assumptions

- D1 takes the board's "a running gate shell holds the turn end" as the expected behaviour.
- D4's envelope for a non-done digest is an error envelope carrying the digest fields, matching `batch_failed`'s shape.
- D6 reads #500's "the plan validator flags such a glyph at Plan time" as: a variant set passes at Plan time, and a genuinely ambiguous answer keeps its Plan-time `glyph-ambiguous` with a detail naming the overlap; a Create target cannot be checked for variants before it exists.
- D7 reads the board's "find whether the check resolves unexported identifiers and identifiers in test files" as answered no-fault by today's quarry, so the fix is the message plus D6.

## Open risks

- A Master that starts a gate shell that never ends holds the run to its `Timeout`; at base the same run would have ended `done` with the result discarded.
- D6's closed GOOS and GOARCH list goes stale with a new toolchain value, which can only make a pair read overlapping, never disjoint wrongly.
- A Create implemented in only some of its variant files resolves `found` and passes, while another platform's build breaks; base behaves the same, and the plan-level verify's build is what catches it.
- `fork-wait-noise` edits the Master template and the implementer body; D5 is limited to its two statements and the one reworded sentence, and whichever task lands second rebases its text.

## Acceptance criteria

- A gated run whose turn end has every output file present and an outstanding shell labelled `cd <dir> && lyx gate test --tags integration ./x | tail -5` keeps waiting; the shell's later turn end with nothing outstanding arrives at the gate.
- The same turn end with only an unrelated unawaited shell outstanding still finishes Done, and the shell lands on `EndedShells`.
- A held gate shell still outstanding at the run's deadline ends the run on the deadline and lands on `EndedShells`.
- A label `cd <dir> && lyx webster recover-batch 3` now matches, and every label base matched by plain prefix still matches.
- A failed `verify` gate entry re-prompts Master with Webster's text naming the findings file and the fixer section, on a fresh, resumed and attached run alike; an entry with no re-prompt function sends `gateRepromptText` unchanged.
- A second recovery's prompt lists a path only the first recovery wrote under "Written by this run", after the first recovery's record was replaced by the respawn; a path no recorded session wrote stays under "Not written by this run".
- A recorded recovery session with no transcript yields a warning and no writes, and the recovery prompt and reset still proceed.
- The recovery session list survives a re-begin and a respawn, a failed start records nothing, and a record written before the field exists reads as an empty list; a batch with no recovery renders the block as at base.
- `recover-batch` exits non-zero with the digest fields on every non-done terminal digest, `dead` with `recovery_retry: true` included, and 0 on `done` and on the running snapshot.
- `tmuxAllowed` declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`) passes its Create done-check.
- `StartTime` in `proc_linux.go` and `proc_windows.go` passes a Plan-time Edit and a Delete glyph check, and its Delete done-check passes only once both are gone.
- Two candidates under `//go:build linux` and `//go:build unix`, or one with an unreadable constraint, stay ambiguous with the overlap named; a non-Go plan's ambiguous answer is unchanged; a Rename of a variant set is still refused.
- A `create-not-done` detail names the key, the status and, for `ambiguous`, every candidate file.
- The implementer body names `lyx gate test` and `lyx loom lint-comments` as allowed and states the tmux and llm `**Verify:**` rule; the Master template's recover-batch rungs read the digest from a non-zero envelope.

## Notes for the plan writer

- D1: `internal/websterengine/runlevel.go` (`masterAwaitedShellPrefix`, the `AwaitedShellPrefixes` spec field) and `internal/shuttleengine/wait.go` (`awaitedLabel`).
- D2: `internal/shuttleengine/gate.go` (`GateEntry`, `gateRepromptText`), its one send in `wait.go`, and the `verify` entry in `runlevel.go`; `websterengine/doc.go` names `NewVerifyGate`, which is `internal/loomshed`'s.
- D3: `internal/websterengine/runwrites.go`, `recoverbatch.go` (`uncommittedPathsBlock`), `reset.go`, `contractevidence.go`, `state.go` (`BatchState`); `claudeengine`'s `AuditForks` errors on a missing parent transcript.
- D4: `internal/webstercli/recoverbatch.go`'s terminal branch.
- D6/D7: `internal/planglyph/donecheck.go`, `resolve.go` (`statusFindings`), `create.go`, `deleteorder.go`, `handle.go`; `internal/impactset/guards.go` and `cmd/testtiming/spawnscan.go` already read `go/build/constraint`.

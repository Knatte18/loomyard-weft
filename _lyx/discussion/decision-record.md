# Decision record — webster-stability

## Goal

Close the Webster faults outside the audit that stop or mislead runs: a gated run that finishes while a background gate shell is still running, Master fixing a verify failure itself, a second recovery disowning the first recovery's files, a recovery prompt that leaves its allowed gates unsaid, a done-check refusal that hides how the target was looked up, and a member declared once per build-constraint set that resolves `ambiguous` and so can never pass a Create or Edit.

## Scope

In scope, one numbered item per board section:

1. Shuttle's turn-end wait for a gated run: a waiting turn end with any background task outstanding, a shell of either signal included, never counts as an arrival while Master's output files exist.
   The run's record of shells it ended over stays and says the shell's result was never read.
2. The verify gate's re-prompt and findings text: both open with Master's way forward, which is to spawn the fixer fork and never fix, edit or commit itself.
3. Recovery attribution: every recovery strand's session is recorded on the batch and audited for write evidence, so an earlier recovery's uncommitted files land under "Written by this run".
   `lyx webster recover-batch` exits non-zero whenever the batch it returns for is terminal and not `done`.
4. The implementer body stencil names the card gate, the batch gate, `lyx loom lint-comments` and `lyx quarry resolve` as always allowed, says the fabric ban covers `lyx fabric` mutating verbs only, and says what to do with a `tmux`- or `llm`-tagged test.
5. The done-check findings (`create-not-done`, `delete-not-done`, `rename-not-done`) say what was looked up and what quarry answered, candidates and their build constraints included.
   The resolver itself has no unexported or test-file limit: at HEAD, `lyx quarry resolve` answers `found` for an unexported function (`internal/planglyph#resolveKeyFor`), a test function (`internal/planglyph#TestCreateFindings_ResolvedTargets`) and the historical target `cmd/testtiming#rusageStats`.
   Both #511 targets were declared per build constraint at the time, so #511 is #500 in another coat.
6. A member whose every declaration sits in files no single build environment selects together is one member: `internal/planglyph` classifies such an `ambiguous` answer as resolved, like `multipart`, in every status policy, so Edit, Delete, Rename, `Uses:` and the Create done-check all accept it.
   A genuinely ambiguous Edit or Delete target stays blocked at Plan time, naming each candidate's file and constraint, so the only ambiguity that can first surface at record-batch is one the batch's own Create introduced.

Out of scope, with the reason:

- Changing quarry itself (it is the external module `github.com/Knatte18/quarry` at a pinned version; the classification lives in lyx, over the candidate files quarry already names).
- Letting a Create handle reuse a name whose every declaration the same card deletes (#507's first option): with per-constraint Edit and Delete working, retiring and re-declaring the bodies is spelled as an Edit, and the resolve pass reading the tree before any card runs stays as designed.
- Reviving a dead batch through `lyx loom goto` (#511's third point): not on the board entry.
- Killing a background shell from lyx: lyx has no handle on a provider's shell process; strand removal is what ends it, as today.
- Ungated runs' turn-end rule: unchanged, since no ungated producer showed the fault and holding a review producer on a stray shell is a cost nobody asked for.

## Decisions

### D1. A gated run's waiting turn end never counts at once

Decision: in `internal/shuttleengine`, the rule that counts a gated run's waiting turn end as Done at once when every outstanding task is an unawaited shell goes away.
A gated run's turn end with any task outstanding, a shell of either signal or a fork, waits until the task's completion starts the next turn, bounded by the run's deadline and the liveness check, exactly as a turn end with an output file missing waits today.
Shells past `background_shell_wait_min` are still logged once and shown in the wait marker; the bound still expires nothing.
`Result.EndedShells`, the `RunResult.Warnings` entry, the `webster-background-shell` friction note and summary.md's "Background shells at the run's end" section stay for the shells a deadline, liveness failure or stop still ends over, and their wording says the shell's result was never read by the run.

Rationale: the at-once count is the one path left after #496 by which a run reads `done` with a gate tier unfinished.
A shell's completion notification starts Master's next turn, so holding is what the fork case already does, with the same bound.
No new edge or weakened guard: the change removes an early exit.

### D2. The verify gate names Master's way forward in the re-prompt and in the findings

Decision: `shuttleengine.GateEntry` gains an optional one-line re-prompt text an entry supplies; the loop sends it instead of the default "read it, fix every finding, and end your turn".
Webster's `verify` entry supplies one that keeps the opening `Gate findings recorded at <path>` (the Master stencil's trigger) and continues: do not fix them yourself; spawn one fixer fork as the gate-failure section of your prompt says, with the prompt file at the rendered verify-fix path; then end your turn.
The verify-gate findings text Master reads opens with the same way-forward line before the attempt count.
The Master stencil's gate-failure section is unchanged in substance.

Rationale: the default re-prompt literally tells Master to fix every finding, and on attempt 3 a `sonnet[medium]` Master obeyed the message over its rules list.
The rule now stands where Master reads the failure, in both the line it receives and the file it opens.
Bound: the custom text is one line with no newline, validated as the default is; an entry without one keeps the default, so no other gated producer changes.

### D3. Recovery sessions are the batch's own, and a non-done recovery exits non-zero

Decision: `BatchState` records the session id of every recovery strand spawned for the batch, read from the shuttle run state at spawn and carried forward across a respawn as `Recoveries` is.
`RunWrites` gains a recovery slice beside Master and forks; `recordedSessions` audits those sessions after the Master sessions.
`writtenWorktreePaths` and the reset's own-path set read all three slices, so a second recovery's "What the worktree holds" lists the first recovery's uncommitted files under "Written by this run".
The contract-file evidence keeps reading the Master slice alone.

`lyx webster recover-batch` exits non-zero, with `ok: false` and the same digest fields (`batch`, `status`, `head_sha`, `deviations`, `dead_reason`, `elapsed_s`, `recoveries`, `recovery_retry`, `warnings`), whenever the record it returns for is terminal and not `done`: `stuck` and `dead` join `failed`.
The message names the ladder's way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, else end the run stuck naming the batch.
A running snapshot under a shorter `--wait` and a `done` digest still exit 0.

Rationale: attribution by shuttle run id disowned nineteen files the run itself wrote; recovery sessions are the run's work by the board's definition.
Bound on the weakened guard: a recovery session's writes join the own-path and written-path sets only, never the Master-wrote-the-contract-file evidence, so a recovery writing `outcome.yaml` is still a finding and not Master's rewrite.
The exit code follows the verb's meaning: the recovery did not recover.
Master reads digest fields, not the exit code, so its ladder is unchanged.

### D4. The implementer body says which commands are always allowed and what a tagged test gets

Decision: the shared implementer body's sandbox paragraph says that the card gate steps, the batch gate steps, `lyx loom lint-comments` and `lyx quarry resolve` are always allowed from any strand and are never a fabric reference, and that the sandbox ban covers mutating `lyx fabric` verbs only.
Its raw-`go` paragraph adds: a `lyx gate test` step runs as written, whatever its `--tags`; a test under the `tmux` or `llm` tag that no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, and running those tiers is left to the gates after webster.

Rationale: the recovery had the rule as a refusal it had not hit yet and a tier table it had never read.
No new edge: `lyx gate test` with any tag is already slotted and allowed from a strand.

### D5. Done-check findings carry the lookup

Decision: a `create-not-done`, `delete-not-done` or `rename-not-done` detail names the resolve key that was looked up (the glyph behind a `plan:` handle), quarry's status, the unit status for `not_found` (unit found and member missing, or unit missing), and for `ambiguous` each candidate's file with its build constraint, the `//go:build` line or filename suffix, or "no constraint".
The refusal-spec "card not done" row says the detail carries the lookup.
No resolver change: the probes above show unexported and test-file identifiers resolve.

Rationale: the refusal must tell a missing symbol from an ambiguity the implementer introduced, which is the only resolver answer left that can first surface at record-batch after D6.

### D6. Constraint-partitioned ambiguity is one member

Decision: `internal/planglyph` gets one classifier over an `ambiguous` answer's candidates: the answer is partitioned when no single build environment selects two candidate files.
The evaluation uses `go/build.Context.MatchFile` per candidate file, which honors `//go:build` lines and `_GOOS`, `_GOARCH` filename suffixes, over an enumerated environment set: GOOS over the OS names the candidates mention plus one they do not, GOARCH the same way, cgo on and off, and every assignment of the other tags their constraint lines mention.
A candidate set naming more than a small cap of other tags, a file that cannot be read, or a constraint that cannot be parsed is not partitioned.
The known GOOS and GOARCH names are one closed list in planglyph.

Every status policy in the package reads an ambiguous answer through the classifier: a partitioned answer is resolved like `multipart` for `statusFindings` (Edit, Delete, Rename endpoints, `Uses:`), DoneChecks' Create and Rename-new direction, `answerSymbols` in `deleteorder.go` (the candidates are the declarations) and the plan-gate passes that resolve a member into its file.
The Delete direction is unchanged: anything but `not_found` still exists.
`createFindings` is unchanged: an ambiguous Create target, partitioned or not, is `create-already-exists`.
`glyph-ambiguous` and the D5 details name each candidate's file and constraint.
The loom plan spec's `glyph-ambiguous` check and planglyph's check-ID list say a partitioned answer resolves.

Rationale: declaring one symbol once per mutually exclusive constraint set is idiomatic Go, and the workaround forced a redesign the card never asked for.
Exact evaluation over the enumerated environments costs a few file reads per ambiguous answer and reuses the standard library's own matcher, so no build-constraint vocabulary is reimplemented.
Bound on the weakened guard: only an answer whose candidates provably never coexist passes; the cap and every read or parse failure fall back to `ambiguous`, and the Delete direction keeps blocking on any surviving declaration.

## Constraints

- `PATTERN-refusal-way-forward`: the non-done recovery exit (D3) is a refusal with a new row in the webster section; the "card not done" row changes wording (D5); every new message names its way forward and a test reaches it.
- `PATTERN-no-denied-recovery`: the new strings in `internal/webster*` and `contracts/stencils/webster/` name `lyx gate test`, `lyx loom lint-comments`, `lyx quarry resolve` and `lyx webster recover-batch` only, never a denied form.
- `PATTERN-completion-signal`: D1 removes a positive exit in `wait.go` and adds no negative verdict; the tripwire test's audited site list is updated in the same card.
- `PATTERN-shuttle-provider-seam`: D1 and D2 touch `shuttleengine` without reading any payload shape; the shell signals stay `claudeengine`'s.
- `PATTERN-glyph-conversion-chokepoint`: D6 reads `quarry.Symbol.File` as the path quarry hands back and never derives a path from a glyph or a glyph from a path.
- `PATTERN-ref-shape-registry`: D6 adds no ref kind and no handle grammar.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: D4 edits the stencil under `contracts/stencils/webster/` only, with its claim pins in `contracts/stencils/claims_test.go`; the Master stencil keeps pointing at the gate-failure section rather than restating the fixer prompt.
- `PATTERN-parent-directive`: no stencil change tells an agent to ask the operator.
- `PATTERN-friction-capture`: the `webster-background-shell` note stays Go-authored; only its wording changes.
- `PATTERN-test-economy` and `PATTERN-test-speed`: every scenario below lands in tier 1 as a new table row or an extended test where one exists; `cmd/lyx/testdata/test-budget.yaml` is listed under `Edit:` by any card whose new top-level tests push `internal/planglyph`, `internal/websterengine`, `internal/shuttleengine` or `internal/webstercli` past its row.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule for every `.md` touched.
- `PATTERN-documentation-lifecycle`: the module docs in `internal/shuttleengine/doc.go`, `internal/websterengine/doc.go` and `internal/planglyph/doc.go` change in the same commit as their code; no `docs/overview.md` change, since no module is added.
- No PATTERN entry is changed or waived.

## Auto-mode assumptions

- Holding over stopping (D1): the board offers either; holding reuses the fork path and the existing deadline bound, and lyx cannot kill a provider shell anyway.
- Ungated runs keep today's at-once count (D1).
- A per-entry re-prompt field over a per-result one (D2): the way forward is a property of the entry, known at wiring time.
- No mechanical gate-time guard against a Master self-fix (D2): the run-exit audit already catches it and a gate-time `Terminal` would end the run no better.
- Recovery writes join the written-path and reset sets but not the contract-file evidence (D3).
- `stuck` and `dead` exit non-zero alongside `failed` (D3): the board names the failed case; the rule "terminal and not done" is the one a reader can predict.
- Compile-only through `lyx gate test ... -- -run '^$'` for tagged tests (D4): a raw tagged `go` run is denied in a strand and the slotted form is already allowed.
- Exact evaluation over enumerated environments with a tag cap, and a closed GOOS and GOARCH list in planglyph (D6), over a cheaper textual rule that would fail open on `linux` beside `amd64`.
- `createFindings` keeps treating a partitioned Create target as `create-already-exists` (D6).

## Open risks

- A transcript-reported shell whose completion notification shuttle never recognizes now holds a gated run until its deadline, where it ended the run `done` before; the run then ends `OutcomeTimeout` with the shell in `EndedShells`.
- The stencil and re-prompt wording (D2, D4) can only be judged by a live run; no `llm`-tier test is added for it.
- An operator script reading `recover-batch`'s exit code sees non-zero for `stuck` and `dead` from now on.
- Auditing recovery sessions adds one `AuditForks` read per recovery spawn to every reader of run writes.
- The environment enumeration's tag cap leaves a rare many-tag candidate set `ambiguous`, which the detail then explains.

## Acceptance criteria

- A gated run whose output files exist at a turn end with one unawaited shell outstanding stays waiting; the next turn end with nothing outstanding arrives at the gate; an ungated run under the same facts still counts at once.
- A gated run held on a shell past the deadline ends `OutcomeTimeout` with the shell in `EndedShells`, and the warning, note and summary section say its result was never read.
- A gate entry with its own re-prompt line is sent that line; an entry without one is sent the default; a multi-line custom line is refused as the default's validation refuses it.
- Webster's `verify` entry's re-prompt opens with `Gate findings recorded at` and names the fixer-fork step and the verify-fix prompt path; the findings file opens with the same way forward.
- A second recovery's prompt lists a path the first recovery strand wrote under "Written by this run" and a path no session wrote under "Not written by this run".
- A recovery strand's write of `outcome.yaml` is still a contract-file finding, not Master's rewrite.
- `recover-batch` exits non-zero with the digest fields for a terminal `stuck` and a terminal `dead` record, with a message naming the retry or the stuck ending by `recovery_retry`; a `done` digest and a running snapshot exit 0.
- The implementer body carries the always-allowed list, the `lyx fabric`-only ban wording and the tagged-test rule, pinned by the claims test; the no-denied-recovery scan passes.
- A `create-not-done` detail for a `not_found` answer names the key and whether the unit exists; for an `ambiguous` answer it names each candidate's file and constraint.
- A member declared in a `//go:build linux` file and a `//go:build !linux` file, and one declared in `_linux.go` and `_windows.go` files, resolves as one member for Edit, Delete, Rename-old, `Uses:` and the Create done-check; declared in `//go:build linux` and `//go:build amd64` files it stays `glyph-ambiguous`, naming both constraints.
- A `//go:build unix` file beside a `_windows.go` file is partitioned; a candidate file that cannot be read or parsed, or a set over the tag cap, is not.
- A partitioned Delete target that still resolves is `delete-not-done`; a partitioned Create target at Plan time is `create-already-exists`.
- `answerSymbols` returns a partitioned answer's candidates, so a later-card reference to one still raises `delete-before-reference`.
- The refusal-spec webster section has the new recovery row and the changed "card not done" row, and each is reached by a test.

## Notes for the plan writer

- D1: `countsAtOnce` and `expiredTurnEnd` in `internal/shuttleengine/wait.go`; the file doc, `internal/shuttleengine/doc.go`, `internal/websterengine/doc.go`'s "Background shells in Master's wait" section, `AppendBackgroundShells` and the note text in `internal/websterengine/runlevel.go`, and the `master-fork-running` corpus case under `internal/shuttleengine/claudeengine/testdata/corpus/`.
- D2: `gateRepromptText` in `internal/shuttleengine/gate.go`, `NewVerifyGate` in `internal/websterengine/verifygate.go`, `renderVerifyGateFindings` in `internal/websterengine/verifygatereport.go`; the verify-fix prompt path is already a Master marker.
- D3: `recoverSpawn` reads the session id from the run state `shuttleengine.FindRun` returns; `recordedSessions`, `loadRunWrites` and `writtenWorktreePaths` in `internal/websterengine/runwrites.go`; the terminal branch of `recoverBatchCmd` in `internal/webstercli/recoverbatch.go` and its `Long` text; the Master stencil's line saying the backgrounded call's output is the terminal digest or a refusal.
- D4: `contracts/stencils/webster/webster-body-implementer.md`, "The sandbox rule" and "Slot-busy exits and raw `go` runs"; pins in `contracts/stencils/claims_test.go`.
- D5 and D6: `internal/planglyph/donecheck.go`, `resolve.go`, `deleteorder.go`, `plangate.go`, `callercoverage.go`; `candidateList` is the one renderer for candidates and can carry the constraint; the GOOS and GOARCH list can be taken from `go tool dist list` once and pinned.
- The loom plan spec's validation-checks section describes `glyph-ambiguous`; `internal/planglyph/doc.go` lists every check ID.

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
6. A member whose every declaration sits in files no single build environment selects together is one member: `internal/planglyph` classifies such an `ambiguous` answer as resolved, like `multipart`, in every status policy, so Edit, Delete, a Rename's new side, `Uses:` and the Create done-check all accept it.
   A Rename's old side stays one declaration, as it is for `multipart`, and its refusal names the way forward for a partitioned member.
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
At the run deadline such a run ends `OutcomeDone` through `classifyDeadlineExpiry`, since its output files exist, and finalize evaluates the gate once more over the tree as it stands; the shell lands in `EndedShells` with its result unread.
Shells past `background_shell_wait_min` are still logged once and shown in the wait marker; the bound still expires nothing.
`Result.EndedShells`, the `RunResult.Warnings` entry, the `webster-background-shell` friction note and summary.md's "Background shells at the run's end" section stay for the shells a deadline, liveness failure or stop still ends over, and their wording says the shell's result was never read by the run.

Rationale: the at-once count is the one path left after #496 by which a run reads `done` with a gate tier unfinished.
A shell's completion notification starts Master's next turn, so holding is what the fork case already does, with the same bound.
No new edge or weakened guard: the change removes an early exit.
Bound: a shell that never reports holds the run to its deadline and never past it.
The deadline still finishes the run `done`, because finalizing a negative verdict over a satisfied file contract is what `PATTERN-completion-signal` forbids;
so the case of a run reading `done` with a shell's result unread remains only after a full deadline, with the gate run once at finalize, and the warning, note and summary say so.

### D2. The verify gate names Master's way forward in the re-prompt and in the findings

Decision: `shuttleengine.GateEntry` gains an optional re-prompt renderer an entry supplies, a function of the findings path returning one line; the loop sends its line instead of the default "read it, fix every finding, and end your turn".
Webster's `verify` entry supplies one that keeps the opening `Gate findings recorded at <path>` (the Master stencil's trigger) and continues: do not fix them yourself; spawn one fixer fork as the gate-failure section of your prompt says, with the prompt file at the rendered verify-fix path; then end your turn.
The verify-gate findings text Master reads opens with the same way-forward line before the attempt count.
The Master stencil's gate-failure section is unchanged in substance.

Rationale: the default re-prompt literally tells Master to fix every finding, and on attempt 3 a `sonnet[medium]` Master obeyed the message over its rules list.
The rule now stands where Master reads the failure, in both the line it receives and the file it opens.
Bound: the rendered line is validated at send time by the default's one-line rule; an entry without one keeps the default, so no other gated producer changes.

### D3. Recovery sessions are the batch's own, and a non-done recovery exits non-zero

Decision: `BatchState` records the session id of every recovery strand spawned for the batch, read from the shuttle run state at spawn and carried forward across a respawn as `Recoveries` is.
`RunWrites` gains a recovery slice beside Master and forks; `recordedSessions` audits those sessions after the Master sessions.
`writtenWorktreePaths` and the reset's own-path set read all three slices, so a second recovery's "What the worktree holds" lists the first recovery's uncommitted files under "Written by this run".
The contract-file evidence counts recovery writes beside fork writes: a contract path's finding clears only when Master's latest successful write to it is later than every fork and recovery write to it.
A reset now discards, without the dirt refusal, a tracked path a recovery session of this run wrote successfully; the exemption stays by path and on successful writes only, and a path no session of this run wrote still refuses.

`lyx webster recover-batch` exits non-zero, with `ok: false` and the same digest fields (`batch`, `status`, `head_sha`, `deviations`, `dead_reason`, `elapsed_s`, `recoveries`, `recovery_retry`, `warnings`), whenever the record it returns for is terminal and not `done`: `stuck` and `dead` join `failed`.
The message names the ladder's way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, else end the run stuck naming the batch.
A running snapshot under a shorter `--wait` and a `done` digest still exit 0.

Rationale: attribution by shuttle run id disowned nineteen files the run itself wrote; recovery sessions are the run's work by the board's definition.
Bound on the weakened guards: a recovery session's writes join the own-path and written-path sets, so a reset may discard what a recovery of this run wrote and nothing else;
on the contract-file evidence they stand with the forks', so a recovery writing `outcome.yaml` after Master's last write keeps the finding rather than clearing as Master's rewrite.
The refusal and audit text for an uncleared contract file, and the refusal-spec rows for a contract file last written by a fork, name the last writer as a fork or a recovery session, so a recovery-kept finding never reads "a fork wrote".
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
No resolver change: quarry already resolves unexported and test-file identifiers.

Rationale: the refusal must tell a missing symbol from an ambiguity the implementer introduced, which is the only resolver answer left that can first surface at record-batch after D6.

### D6. Constraint-partitioned ambiguity is one member

Decision: `internal/planglyph` gets one classifier over an `ambiguous` answer's candidates: the answer is partitioned when no single build environment selects two candidate files.
The evaluation uses `go/build.Context.MatchFile` per candidate file, which honors `//go:build` lines and `_GOOS`, `_GOARCH` filename suffixes, over an enumerated environment set: every GOOS and GOARCH pair in one closed list in planglyph, taken once from `go tool dist list` and pinned, cgo on and off, and every assignment of the free tags the candidates' constraint lines mention.
Each environment is a zero `build.Context` carrying only `GOOS`, `GOARCH`, `CgoEnabled`, the free tags assigned true in `BuildTags`, and an `OpenFile` serving the candidate's bytes; `Compiler`, `ReleaseTags` and `ToolTags` stay empty, so `gc`, `gccgo`, `go1.N`, `goexperiment.*` and `amd64.vN` tags are free tags evaluated both ways and never fixed true by the toolchain lyx was built with.
`boringcrypto`, which the matcher looks up as `goexperiment.boringcrypto`, is assigned under that name.
`unix` and the GOOS implications (`android` selecting `linux`, `illumos` selecting `solaris`, `ios` selecting `darwin`) come from the GOOS through the matcher and are never assigned as free tags;
a free tag is a mentioned tag that is no GOOS, GOARCH, `unix` or `cgo` name.
Each candidate file is read once and its bytes served to every environment.
A candidate set naming more than four free tags, a file that cannot be read, or a constraint that cannot be parsed is not partitioned.

Every status policy in the package reads an ambiguous answer through the classifier: a partitioned answer is resolved like `multipart` for `statusFindings` (Edit, Delete, Rename endpoints, `Uses:`), DoneChecks' Create and Rename-new direction, `answerSymbols` in `deleteorder.go` (the candidates are the declarations) and the plan-gate passes that resolve a member into its file.
A Rename's old side keeps `renameDeclSource`'s Found-only rule: the new declaration is derived from exactly one signature, and partitioned declarations may differ, as `multipart` ones may.
For a partitioned old side the `rename-old-unresolved` detail names each candidate's file and constraint and the way forward: a Delete of the old member and one Create declaration of the new member's handle on a card that edits each of the old member's files, or an Edit of each file's body.
One declaration head serves every file even when the per-file signatures differ, since the head only computes the new glyph, and partitioned declarations share one glyph; a Create per file is never the way forward, because a second claim on the handle is `handle-collision`.
The Delete direction is unchanged: anything but `not_found` still exists.
`createFindings` is unchanged: an ambiguous Create target, partitioned or not, is `create-already-exists`.
`glyph-ambiguous` and the D5 details name each candidate's file and constraint.
The loom plan spec's `glyph-ambiguous` check and planglyph's check-ID list say a partitioned answer resolves.

Rationale: declaring one symbol once per mutually exclusive constraint set is idiomatic Go, and the workaround forced a redesign the card never asked for.
Exact evaluation over the enumerated environments costs one read per candidate file and in-memory matcher calls, at most the pinned pairs times two cgo values times sixteen tag assignments per candidate, and reuses the standard library's own matcher, so no build-constraint vocabulary is reimplemented.
Bound on the weakened guard: only an answer whose candidates provably never coexist passes; the cap and every read or parse failure fall back to `ambiguous`, no tag is held true by the building toolchain, a Rename's old side never accepts several declarations, and the Delete direction keeps blocking on any surviving declaration.

## Constraints

- `PATTERN-refusal-way-forward`: the non-done recovery exit (D3) is a refusal with a new row in the webster section; the "card not done" row (D5) and the contract-file-last-written-by-a-fork rows (D3) change wording; the `rename-old-unresolved` detail gains its way forward for a partitioned old side (D6); every new message names its way forward and a test reaches it.
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
- Recovery writes join the written-path and reset sets and count with fork writes in the contract-file evidence (D3).
- `stuck` and `dead` exit non-zero alongside `failed` (D3): the board names the failed case; the rule "terminal and not done" is the one a reader can predict.
- Compile-only through `lyx gate test ... -- -run '^$'` for tagged tests (D4): a raw tagged `go` run is denied in a strand and the slotted form is already allowed.
- Exact evaluation over every pinned GOOS and GOARCH pair with a free-tag cap (D6), over a cheaper textual rule that would fail open on `linux` beside `amd64`.
- `createFindings` keeps treating a partitioned Create target as `create-already-exists` (D6).
- A Rename's old side stays Found-only for a partitioned member (D6), as for `multipart`.
- A free-tag cap of four (D6).

## Open risks

- A transcript-reported shell whose completion notification shuttle never recognizes now holds a gated run until its deadline, where it ended the run `done` at once before; the run then ends `OutcomeDone` with the gate run once at finalize and the shell in `EndedShells`, its result unread.
- The stencil and re-prompt wording (D2, D4) can only be judged by a live run; no `llm`-tier test is added for it.
- An operator script reading `recover-batch`'s exit code sees non-zero for `stuck` and `dead` from now on.
- Auditing recovery sessions adds one `AuditForks` read per recovery spawn to every reader of run writes.
- The environment enumeration's tag cap leaves a candidate set naming five or more free tags `ambiguous`, which the detail then explains; with `go1.N` and `gc` counted as free tags, a set mixing version and OS-feature tags reaches the cap sooner.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d6-rename-old-way-forward-per-file-create (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - answersymbols-classifier-rerun-per-call (class design, severity MEDIUM, open in rounds 2)
  - card2-warning-drops-shell-label-and-webstercli-test (class scope, severity MEDIUM, open in rounds 2)
  - card4-run-reprompt-wiring-unpinned (class scope, severity MEDIUM, open in rounds 2)
  - card6-recover-batch-prefix-still-says-fork (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - partitioned-member-consumers-uncovered (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A gated run whose output files exist at a turn end with one unawaited shell outstanding stays waiting; the next turn end with nothing outstanding arrives at the gate; an ungated run under the same facts still counts at once.
- A gated run held on a shell to the deadline ends `OutcomeDone` with the gate evaluated once at finalize and the shell in `EndedShells`, and the warning, note and summary section say its result was never read.
- A gate entry with its own re-prompt line is sent that line; an entry without one is sent the default; a multi-line custom line is refused as the default's validation refuses it.
- Webster's `verify` entry's re-prompt opens with `Gate findings recorded at` and names the fixer-fork step and the verify-fix prompt path; the findings file opens with the same way forward.
- A second recovery's prompt lists a path the first recovery strand wrote under "Written by this run" and a path no session wrote under "Not written by this run".
- A recovery strand's write of `outcome.yaml` after Master's last write to it leaves the contract-file finding standing, and its refusal says a recovery session wrote the file; a fork's write still says a fork wrote it.
- A reset over a tracked path a recovery strand of the run wrote discards it without the dirt refusal; a dirty tracked path no session of the run wrote still refuses.
- `recover-batch` exits non-zero with the digest fields for a terminal `stuck` and a terminal `dead` record, with a message naming the retry or the stuck ending by `recovery_retry`; a `done` digest and a running snapshot exit 0.
- The implementer body carries the always-allowed list, the `lyx fabric`-only ban wording and the tagged-test rule, pinned by the claims test; the no-denied-recovery scan passes.
- A `create-not-done` detail for a `not_found` answer names the key and whether the unit exists; for an `ambiguous` answer it names each candidate's file and constraint.
- A member declared in a `//go:build linux` file and a `//go:build !linux` file, and one declared in `_linux.go` and `_windows.go` files, resolves as one member for Edit, Delete, `Uses:`, the Create and Rename-new done-checks and a plan-gate member read into its file; declared in `//go:build linux` and `//go:build amd64` files it stays `glyph-ambiguous`, naming both constraints.
- A Rename whose old side is a partitioned member is refused `rename-old-unresolved`, naming each candidate's file and constraint and the way forward: a Delete of the old member with one Create declaration of the new handle on a card editing each candidate file, or an Edit of each file's body.
- A `//go:build unix` file beside a `_windows.go` file is partitioned; a `//go:build !windows` file beside a `//go:build unix` file stays `glyph-ambiguous`; a `//go:build !go1.21` file and a `//go:build !gc` file, each beside an unconstrained file, stay `glyph-ambiguous`; a candidate file that cannot be read or parsed, or a set naming five free tags, is not partitioned.
- A partitioned Delete target that still resolves is `delete-not-done`; a partitioned Create target at Plan time is `create-already-exists`.
- `answerSymbols` returns a partitioned answer's candidates, so a later-card reference to one still raises `delete-before-reference`.
- The refusal-spec webster section has the new recovery row and the changed "card not done" row, and each is reached by a test.

## Notes for the plan writer

- D1: `countsAtOnce` and `expiredTurnEnd` in `internal/shuttleengine/wait.go`; the file doc, `internal/shuttleengine/doc.go`, `internal/websterengine/doc.go`'s "Background shells in Master's wait" section, `AppendBackgroundShells` and the note text in `internal/websterengine/runlevel.go`, and the `master-fork-running` corpus case under `internal/shuttleengine/claudeengine/testdata/corpus/`.
- D2: `gateRepromptText` in `internal/shuttleengine/gate.go`, `NewVerifyGate` in `internal/websterengine/verifygate.go`, `renderVerifyGateFindings` in `internal/websterengine/verifygatereport.go`; the verify-fix prompt path is already a Master marker.
- D3: `recoverSpawn` reads the session id from the run state `shuttleengine.FindRun` returns; `recordedSessions`, `loadRunWrites` and `writtenWorktreePaths` in `internal/websterengine/runwrites.go`; the terminal branch of `recoverBatchCmd` in `internal/webstercli/recoverbatch.go` and its `Long` text; the Master stencil's line saying the backgrounded call's output is the terminal digest or a refusal.
- D4: `contracts/stencils/webster/webster-body-implementer.md`, "The sandbox rule" and "Slot-busy exits and raw `go` runs"; pins in `contracts/stencils/claims_test.go`.
- D5 and D6: `internal/planglyph/donecheck.go`, `resolve.go`, `deleteorder.go`, `plangate.go`, `callercoverage.go`, and `renameDeclSource` in `handle.go`; `candidateList` is the one renderer for candidates and can carry the constraint.
- D3's contract-file wording: `contractDeleteClause` in `internal/websterengine/contractevidence.go` and the `fork-contract-write` finding text; the "contract file last written by a fork" rows in `contracts/specs/refusal-spec.md`.
- The loom plan spec's validation-checks section describes `glyph-ambiguous`; `internal/planglyph/doc.go` lists every check ID.

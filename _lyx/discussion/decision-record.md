# Decision record — dbench-n0om: Webster stability

## Goal

Close the Webster faults that stop or mislead runs outside the audit: a background gate shell that ends unobserved while the run reads `done`, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that does not name its allowed gates, a `recover-batch` that exits 0 on a failed terminal batch, and Create, Edit and Delete checks that can never pass for a member declared once per build-constraint set.

## Scope

In:

1. Master's turn end is held by a running `lyx gate` background shell (issues #505, #509, #517), and the record of a shell outstanding at the run's end says its result was not observed.
2. The verify gate's re-prompt to Master names the fixer-fork route instead of "fix every finding" (board item 2).
3. Recovery strands' own writes count as the run's writes, so a second recovery keeps the first one's work (#522).
4. `recover-batch` exits non-zero on every terminal digest that is not `done` (#522).
5. The recovery prompt names the gates it may always run, and what to do with a `tmux`- or `llm`-tagged step (board item 4).
6. A build-split member, one name declared in several files whose build constraints are pairwise disjoint, counts as one declaration in every planglyph status check, at Plan time and at record-batch (#500, #507, #511).
7. The `create-not-done` and `rename-not-done` details name how the target was looked up and what quarry answered (#511).

Out:

- Reviving a dead batch through `lyx loom goto` (#511's third point); it is not on this board entry.
- A `Create` handle reusing a name whose every declaration the same card deletes (#507's first alternative); item 6 makes such a symbol editable in place, which removes the need.
- Changing quarry (`github.com/Knatte18/quarry`); its resolver keeps ignoring build constraints by design.
- A lyx verb that spawns the verify fixer; Master's forks are in-session Agent-tool forks, which no CLI verb can start.
- A Rename whose old side is a build-split member: `renameSignature` still needs exactly one declaration.

## Decisions

### D1 — A `lyx gate` background shell holds Master's turn end

Decision: `websterengine` adds the prefix `lyx gate ` to Master's `AwaitedShellPrefixes`, beside `masterAwaitedShellPrefix`, so a Master background shell whose command starts with it holds a turn end exactly as a fork does, even with both output files present.
Bound: the hold lasts until the shell's completion notification starts Master's next turn, or until the run's own `Timeout` or the liveness check ends the run, which then ends as timeout or died, never `done`.
A gate run whose command does not start with `lyx gate ` (a `cd … &&` prefix, a wrapper script) is not held and falls to D2's record.
Rationale: the provider reports no shell pid, so lyx cannot stop a shell; waiting is the only way its result is observed, and shuttle already has the caller-data seam for it, so no provider knowledge moves (`PATTERN-shuttle-provider-seam`).

### D2 — An unobserved shell is named as unobserved

Decision: the per-shell `webster-background-shell` friction note, its `RunResult.Warnings` entry and the summary's "Background shells at the run's end" section each say the shell's result was not observed by the run.
Rationale: the outcome must never read as "tests passed" when a test shell was cut off; the wording is shared by the note and the warning today, so one change covers both.

### D3 — The verify gate's re-prompt routes Master to the fixer fork

Decision: `shuttleengine.GateEntry` gains an optional per-entry re-prompt text function taking the findings path; an entry without one keeps `gateRepromptText`.
Webster's `verify` entry supplies one that names the findings path, tells Master not to fix the failure itself, and to spawn the one fixer fork its instructions describe, then end its turn.
The text stays one line, as `validateSendText` requires, and the `Gate findings recorded at` opening Master's stencil keys on stays.
The Master stencil's "A gate failure: spawn one fixer fork" section stays the rule's statement; its "Do NOT localize or fix the failure yourself" line moves to the section's first line if it is not already there.
Bound: the function changes only the text sent; attempts, pending and terminal handling stay as they are.
Rationale: the generic text "fix every finding, and end your turn" is the last instruction Master reads before acting and contradicts its stencil, which is the likeliest cause of the `driver-context` self-fix.

### D4 — Recovery strands' writes are the run's writes

Decision: every recovery spawn records its provider session id on the batch record in a list carried onto each fresh record, as `Recoveries` is.
`loadRunWrites` reads those sessions into their own `RunWrites` bucket, beside `Master` and `Forks`, and `writtenWorktreePaths` includes it, so the recovery prompt's "Written by this run" group, and `PlanReset`'s own-path set, count a path an earlier recovery wrote.
No audit that judges Master's writes reads the new bucket, so a recovery's writes are never a parent-write finding.
Bound: `PlanReset`'s dirty-path guard now lets through a dirty tracked path that a recorded recovery session of this run wrote successfully, and nothing else; a recovery whose session id was never recorded stays unattributed, and its paths stay "Not written by this run".
Rationale: attribution by batch keeps the first recovery's work as the second's starting point without a destructive reset of the tree; the strand already checks such work against the card and the suspect rule.

### D5 — `recover-batch` exits non-zero on a failed terminal digest

Decision: a terminal digest whose status is `stuck`, `dead` or `failed` exits non-zero, carrying the same digest fields (`recoveries`, `recovery_retry` included) plus an error naming the way forward the failure ladder already gives for that status: `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise end the run stuck naming the batch.
A `done` digest and a running snapshot keep exit 0.
The refusal lands with its row in `contracts/specs/refusal-spec.md` and a test reaching it (`PATTERN-refusal-way-forward`), and names no denied command (`PATTERN-no-denied-recovery`).
Master's stencil rungs, which read the envelope, are checked to read the digest from a non-zero exit too.
Rationale: an exit 0 on a batch that ended failed reads as success to any caller that checks the exit code.

### D6 — The recovery prompt names its allowed gates

Decision: `webster-prefix-recovery.md` states that the card gates, each card's `verify:` lines, the batch gate, `lyx gate test` and `lyx loom lint-comments` are always allowed, and that the sandbox rule's ban covers only mutating `lyx fabric` verbs.
It states that a `tmux`- or `llm`-tagged step the strand cannot run is left to the plan-level verify: the strand compile-checks it, names it in its final reply, and does not count it as a gate failure.
Rationale: the recovery strand, starting cold, could not tell whether those gates fell under the fabric ban; the fork inherits Master's context and is not changed.

### D7 — A build-split member counts as one declaration

Decision: `internal/planglyph` gets one classifier over a quarry `ambiguous` answer: the answer is build-split when it has at least two candidates, each in a different file, and the files' build constraints are pairwise disjoint.
A file's constraint is its `//go:build` line and its filename's GOOS and GOARCH suffixes, parsed with `go/build/constraint`.
Two constraints are disjoint when no configuration satisfies both, where a configuration sets exactly one GOOS and one GOARCH, derives `unix` from the GOOS, and sets every other tag freely.
Every planglyph reader of a resolve status treats a build-split answer as it treats `multipart`: `statusFindings` passes it for an Edit or Uses target, `DoneChecks` passes it for a Create target and a Rename's new side and keeps blocking a Delete or a Rename's old side, `create-already-exists` keeps blocking it, and `answerSymbols` returns its candidates as declarations.
Bound: the classifier lets through only an ambiguous answer it proves disjoint; an unparsable constraint, a candidate without a file, two candidates in one file, or a configuration space past a fixed atom bound is not build-split and keeps today's verdict, `glyph-ambiguous` or `create-not-done`.
Rationale: quarry deliberately reports both candidates rather than guess a GOOS, so the policy belongs in lyx's status policy; one classifier read by every check means a glyph Plan accepts cannot fail record-batch for this reason, which is the board's "flag at Plan time".

### D8 — Done-check details say how the target was looked up

Decision: the `create-not-done` and `rename-not-done` details name the resolved key, quarry's status, the unit it searched, and for an `ambiguous` answer each candidate's file with the build-split verdict and its reason.
The detail states that quarry reads every `.go` file of the unit, test files and unexported names included, and does not evaluate build constraints.
Rationale: #511's operator could not tell a missing symbol from a resolver limit; quarry resolves unexported and same-package test-file names already, so the likely cause there was an ambiguous answer the detail hid.

### D9 — Docs land with the code

Decision: `internal/websterengine/doc.go` (background shells, recovery attribution, recover-batch exit), `internal/shuttleengine/doc.go` (per-entry re-prompt text), `internal/planglyph/doc.go` (build-split), `contracts/specs/webster-spec.md` where it states recover-batch's exit or the gate re-prompt, and `refusal-spec.md` (D5) change in the same task.
Rationale: CLAUDE.md's "docs land in the same commit" rule.

## Constraints

- `PATTERN-refusal-way-forward`: D5's new non-zero exit is a refusal in a bound module; its row and a reaching test land with it.
- `PATTERN-no-denied-recovery`: D5's way forward and D6's stencil text name no denied form.
- `PATTERN-shuttle-provider-seam`: D1 and D3 put no provider specifics in `shuttleengine` or `websterengine`; the prefix and the text are caller data.
- `PATTERN-stencil-ownership`: D3's and D6's stencil text lives in `contracts/stencils/`, read from disk.
- `PATTERN-glyph-conversion-chokepoint`: D7 reads a candidate's file from the path quarry's answer carries; it converts no glyph to a path itself.
- `PATTERN-sole-parsers`: the plan is still read only through `planparser`.
- `PATTERN-test-economy`, `PATTERN-test-speed`: each behaviour is pinned at its package's public surface, extending existing table tests where they exist; D7's classifier is pure and tested on fixtures without git where possible.
- `PATTERN-test-isolation`: a planglyph test that builds a repository runs under the package's existing harness.
- CLAUDE.md test budget: a card whose new top-level tests push a package past `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Comments and markdown use semantic line breaks (`PATTERN-comment-line-breaks`).

## Auto-mode assumptions

- A1: Master's `lyx gate` shells were Master's own (D1); a fork's background shell does not surface in Master's Stop payload as Master's. If the plan writer finds the cut-off shells belonged to the fixer fork, D1 still holds, since the prefix matches whatever task shuttle reports.
- A2: the `driver-context` self-fix was triggered by the generic gate re-prompt text (D3); the run's transcript was not read.
- A3: "Publish verify attempt 3" in the board entry means the verify gate's third attempt on Master's run.
- A4: #511's two targets were build-split when the batches failed; the current tree declares `tmuxAllowed` once, so the failing tree is gone. D8's detail exists so the next case shows its cause.
- A5: the recovery provider's session id is readable at spawn from the shuttle run the spawn starts.
- A6: the recovery strand's inability to run `tmux`-tagged steps is a settings or nesting limit the stencil should describe, not a bug this task fixes.

## Open risks

- R1: D1 holds a turn end for a long integration run; a gate shell that hangs holds the run until its `Timeout`. The run then ends timeout, which is the correct reading.
- R2: D7's configuration space needs the known GOOS and GOARCH values and the `unix` set; a hand-kept list goes stale as Go adds ports. The plan should derive them from the toolchain where it can, and fail closed (not build-split) on an unknown value.
- R3: D5 changes an exit code Master's backgrounded call reports; if a stencil rung keyed on exit 0 for a stuck digest, it must change in the same card.
- R4: D4 widens `PlanReset`'s own-path set; a recovery that wrote a path and was then reset past keeps that path classed as the run's own.

## Acceptance criteria

- A Master background shell whose command starts with `lyx gate ` holds a gated turn end with both output files present, and the run ends `done` only after the shell's completion and a later turn end.
- A Master shell outstanding at the run's end yields a warning and friction note stating its result was not observed.
- The verify gate's re-prompt to Master names the findings path and the fixer-fork route and does not say to fix the findings; any other gate entry still sends the default text.
- A second recovery of a batch lists the paths the first recovery wrote under "Written by this run".
- A recovery's writes produce no parent-write audit finding.
- `PlanReset` accepts a dirty tracked path a recorded recovery session wrote, and still refuses one no session of the run wrote.
- `recover-batch` exits non-zero with the digest and a way forward on a `stuck`, `dead` or `failed` terminal digest, and exits 0 on `done` and on a running snapshot; the refusal has a row in `refusal-spec.md`.
- The recovery prompt names the always-allowed gates and the `tmux`/`llm` step rule; the edit and parent directives still render.
- A member declared in two files with disjoint constraints (`//go:build tmux || llm` and `//go:build !tmux && !llm`; `_linux.go` and `_windows.go`; `//go:build linux` and `//go:build !linux`) passes a Create done-check and an Edit status check, and still blocks a Delete done-check and `create-already-exists`.
- Two candidates whose constraints overlap (`linux` and `unix`; no constraint on either; both in one file) keep `glyph-ambiguous` at Plan time and `create-not-done` at record-batch.
- An unparsable constraint or an unknown GOOS on a candidate is not build-split.
- A `create-not-done` detail names quarry's status and unit, and for an ambiguous answer each candidate's file and the build-split verdict.

## Notes for the plan writer

- Turn-end hold rules: `internal/shuttleengine/doc.go` ("A turn end that leaves background work outstanding"), `awaitedLabel` and `expiredTurnEnd` in `wait.go`; Master's spawn spec sets `AwaitedShellPrefixes` in `internal/websterengine/runlevel.go`.
- Shell labels come from the Bash command, else the description (`claudeengine/background.go`).
- Re-prompt text: `gateRepromptText` in `internal/shuttleengine/gate.go`; webster's verify entry is built in `runlevel.go`.
- Run writes: `loadRunWrites`, `recordedSessions` and `writtenWorktreePaths` in `internal/websterengine/runwrites.go`; recovery prompt grouping in `recoverbatch.go`; batch record fields in `state.go`.
- Status readers to route through the classifier: `statusFindings` (`resolve.go`), `createFindings` (`create.go`), `doneCheckVerdicts` (`donecheck.go`), `answerSymbols` (`deleteorder.go`).
- quarry's rationale for ignoring build constraints is `statusForMatches` in its `internal/engine/resolve.go`.

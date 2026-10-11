# Decision record — dbench-omsm

## Goal

Close the Webster faults that stop or mislead runs outside the audit:
a background gate shell cut off at Master's turn end,
Master fixing a verify failure itself,
a second recovery treating the first recovery's files as foreign,
a recovery prompt that does not name its allowed gates,
and Create, Edit and Delete checks that cannot pass for members declared once per build-constraint set or per GOOS.

## Scope

In:

1. Master's turn-end wait holds on a running `lyx gate` shell (board item 1, #505, #509, #517).
2. The verify-gate re-prompt Master receives states the fixer-fork rule (board item 2).
3. Recovery write attribution by batch, and `recover-batch` exit status on a failed terminal digest (board item 3, #522).
4. The recovery prompt names its allowed gates and leaves tmux-tagged steps to verify (board item 4).
5. The create-not-done refusal says how the target was looked up and what quarry answered (board item 5, #511).
6. planglyph treats an ambiguous answer whose candidates sit in mutually exclusive build-constraint files as one declaration, and lets a Create reuse a name whose every declaration the plan deletes earlier (board item 6, #500, #507).

Out:

- Changing quarry (an external module, `github.com/Knatte18/quarry`); every fix here lives in loomyard.
- The halt at a dead batch after `lyx loom goto` into Webster (#511's last paragraph); see `## Open risks`.
- A verb that spawns the verify fixer: forks are the Agent tool inside Master's session, which no `lyx` verb can start.

## Decisions

### D1 — A running `lyx gate` shell holds Master's turn end

Master's spawn spec declares `lyx gate` as a second awaited shell prefix beside `lyx webster recover-batch`, so shuttle's turn-end wait treats a running `lyx gate …` background shell like a fork: the turn end does not count while it runs, whether or not Master's output files exist.
Master's stencil gains one line, in its verify-gate section: Master never runs `lyx gate` itself; the gate runs after its turn.
Every other background shell keeps the existing ended-shell record (friction note, `RunResult.Warnings`, summary section).

Rationale: the hold is the board's first expected behavior and reuses the existing awaited-prefix mechanism, so the outcome can no longer read "done" over an unfinished gate tier.
The stencil line removes the reason Master starts such a shell; the prefix is the backstop.

Bound: the hold lets a turn end wait past `background_shell_wait_min` for a `lyx gate` shell only;
it is bounded by the run's deadline and the liveness check, exactly as the recovery prefix is.
The prefix matches on the shell label's start, so a shell whose command does not begin with `lyx gate` (for example `cd x && lyx gate …`) is not held and falls back to the ended-shell record.

### D2 — Webster supplies its own verify-gate re-prompt

Shuttle's gate spec gains an optional per-run re-prompt text; when empty, shuttle sends today's text.
Webster sets it for Master's verify gate to a single-line text that names the findings file, says not to fix the findings itself, and points at its stencil section "A gate failure: spawn one fixer fork" and the rendered `verify_fix_prompt_path`, ending with: spawn exactly one fixer fork, then end the turn.
Master's stencil section stays the rule's statement; the re-prompt points at it and does not paraphrase the fork prompt (`PATTERN-producer-pointer-rule`).
The wording is owned by webster as a Go string handed to shuttle;
if it is instead taken from `contracts/stencils`, it is read through `internal/stencilstore` (`PATTERN-stencil-ownership`).

Rationale: the generic text "read it, fix every finding, and end your turn" arrives at the failure and contradicts the stencil, and the later, nearer message wins; `driver-context`'s self-fix followed it.
Other producers keep their current text.

### D3 — Recovery write evidence is attributed by batch

A batch's state keeps every session id that ever worked it, the fork's and each recovery's, appended and never overwritten.
`loadRunWrites` reads every recorded session, so a second recovery's prompt lists files an earlier recovery of the same batch wrote under "Written by this run".
A `state.json` written before the field exists still loads, its single recorded session read as before.

Rationale: the batch's single session id is overwritten by each recovery, so the first recovery's writes lose their evidence and fall under "Not written by this run".
Attribution by batch keeps the earlier attempt's work as the starting point the recovery prefix already describes, with no destructive reset.

### D4 — `recover-batch` exits non-zero on a failed terminal digest

`lyx webster recover-batch` exits 0 only when its terminal digest is done.
A stuck or dead terminal digest still prints the full digest JSON on stdout and exits non-zero.
A refusal keeps its own JSON error envelope, so a failed digest and a refusal stay distinguishable by their output (`PATTERN-cli-cobra`).
Master's failure-ladder rungs key on the digest's content, not the exit status;
the stencil changes only if a rung relies on exit 0.

Rationale: #522's `recover-batch 1` exited 0 while the batch ended failed and terminal, which reads as success to anyone checking the exit status.

### D5 — The recovery prompt names its allowed gates

The recovery prefix states that the card gate steps and the batch gate it renders, and `lyx loom lint-comments` among them, are always allowed and are not fabric verbs, rendered from the same source as the card and batch gates rather than hard-coded.
It states that a tmux-tagged gate step is left to the verify gate: the recovery strand compile-checks it and does not count it as a failure.
The shared implementer body is unchanged;
forks carry no ban the gates could be confused with.

Rationale: the recovery prefix's ban on `lyx webster` and the shared sandbox rule left the strand unsure whether gate verbs fell under them.
Naming no command the agents' settings deny keeps `PATTERN-no-denied-recovery`.

### D6 — create-not-done says how the target was looked up

quarry already resolves unexported members and members declared in same-package `_test.go` files, so no resolver change is made for them.
The create-not-done Detail names quarry's answer:
for not_found, that a quarry resolve over the working tree, test files and build-tagged files included, found no declaration;
for ambiguous, every candidate with its file (through planglyph's existing candidate renderer) and that their build constraints are not mutually exclusive.
The `contracts/specs` row naming create-not-done is updated in the same commit (`PATTERN-refusal-way-forward`).

Rationale: #511's targets were most likely ambiguous by build tag or GOOS (the #500 and #507 shape), which the old message "still does not resolve" reported as a missing symbol;
naming the lookup tells a missing symbol from a resolver limit.

### D7 — A disjoint-ambiguous answer counts as one declaration

planglyph decides whether an ambiguous answer is disjoint: no build context makes two of its candidates' files match at once.
The decision enumerates contexts and lets `go/build`'s `Context.MatchFile` judge each candidate file, which honours `//go:build` lines, `_GOOS`/`_GOARCH` filename suffixes and `unix`.
The context domain is the GOOS and GOARCH names the candidates mention, in file names or constraints, plus one value standing for every other;
cgo on and off;
and every subset of the custom tags the candidates' constraints mention.
One planglyph helper maps a resolve answer to its effective status, a disjoint ambiguous answer reading as found with every candidate as its declarations, and every planglyph reader of a resolve status goes through it, so the plan gate and record-batch's done checks cannot disagree.
Effects: a Create done check passes;
an Edit or Delete target is accepted at the plan gate, with no glyph-ambiguous finding;
delete-not-done blocks while any candidate remains;
create-already-exists still fires for a Create of a name that already exists disjoint-ambiguous, unless D8 waives it.
A non-disjoint ambiguous answer keeps raising glyph-ambiguous at the plan gate, so the record-batch surprise moves to Plan time.

Rationale: quarry by design answers ambiguous for a member declared once per build-constraint set or per GOOS, and the idiomatic Go spelling of such a member can then never pass a done check (#500, #507).
Independent tag booleans cannot model GOOS exclusivity, so `foo_linux.go` and `foo_windows.go` would never read as disjoint; `MatchFile` over real contexts can.

Bound: the helper fails closed, the answer staying ambiguous, when any candidate's file is unknown or unreadable, when two candidates share a file, when a candidate's file carries no constraint, or when the candidates mention more custom tags than a small fixed cap (8).
It widens nothing for a non-ambiguous answer.

### D8 — A Create may reuse a name the plan deletes earlier

create-already-exists is waived for a Create whose name's every existing declaration is removed earlier in plan execution order, or in the same card, by a Delete of the candidate's file by path, a glyph Delete of the name, or a Rename's old side.
Execution order is the order planglyph's delete-ordering logic already derives, not card number.

Rationale: #507's card deleted every declaration of a per-GOOS name and created it again;
the plan was coherent, but the gate read the tree before any card ran and forced a rename the design never asked for.

Bound: the waiver needs every candidate covered; one surviving candidate keeps the finding.
It applies only to create-already-exists, never to a done check, so record-batch still verifies the Create landed.

## Constraints

- `PATTERN-completion-signal`: D1 changes only which shells hold a turn end; a negative "did this run finish" answer still consults `allOutputFilesExist` first.
- `PATTERN-shuttle-provider-seam`: the awaited prefix and re-prompt text are provider-neutral spec fields; nothing provider-specific moves out of `claudeengine`.
- `PATTERN-producer-pointer-rule` and `PATTERN-stencil-ownership`: D2's re-prompt points at the stencil section; stencil text is read through `stencilstore`.
- `PATTERN-refusal-way-forward`: D4's non-zero exit and D6's changed Detail keep their way forward, their spec rows land in the same commit, and a test reaches each.
- `PATTERN-no-denied-recovery`: D5 names no command the agents' settings deny.
- `PATTERN-gate-slot`: a gate Master might run is unwanted anyway, which D1's stencil line states.
- `PATTERN-sole-parsers`: planglyph changes stay behind `planparser` for plan reading.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend existing tables (gate re-prompt, awaited prefix, runwrites, donecheck, create, status-completeness) before adding top-level tests; new top-level tests that push a package past `cmd/lyx/testdata/test-budget.yaml` list that file under `Edit:`.
- `PATTERN-comment-line-breaks` and the docs rule: `internal/shuttleengine/doc.go`, `internal/websterengine/doc.go` and `internal/planglyph/doc.go` change in the same commit as the behavior they describe.

## Auto-mode assumptions

- No operator answered; every pick was the chair's, weighed against advisor `ly:dbench-omsm:discussion-advisor-1`, which agreed with each, with refinements adopted in D1, D2, D6, D7 and D8.
- #511's targets were ambiguous by build constraint or GOOS rather than missing; the record says "likely" because the run's tree was not reproduced.
- Master's backgrounded `lyx gate` command labels begin with `lyx gate`; the Plan producer confirms the label claudeengine records.
- A non-zero `recover-batch` exit changes only the completion notification's label in Master's session, since no Master rung keys on exit status.

## Open risks

- A dead batch after `lyx loom goto` into Webster halts at once with a way forward that cannot be taken (#511's last paragraph); out of scope, and possibly a pre-existing `PATTERN-refusal-way-forward` gap.
- D1's prefix match misses a gate shell spelled behind `cd` or `timeout`; the ended-shell record still reports it.
- D7's context domain could miss a platform whose name no candidate mentions but which some file matches implicitly; the "other" value covers unmentioned names, and the tag cap fails closed.
- D3 changes the `state.json` shape; an older binary reading a newer state ignores the new field and sees only the latest session.

## Acceptance criteria

- A Master turn end with output files written and a `lyx gate …` background shell running does not count; the run ends only after the shell finishes, its deadline or the liveness check.
- A Master turn end with a non-gate background shell keeps today's behavior and record.
- A webster verify-gate failure re-prompts Master with the webster text naming the findings file, the stencil section and the fixer prompt path; another producer's gate failure still gets the generic text; the webster text is one line.
- Master's stencil states it never runs `lyx gate`.
- A second recovery of a batch, after a first recovery wrote uncommitted files, lists those files under "Written by this run".
- A `state.json` without the new per-batch session list loads and attributes as before.
- `recover-batch` exits 0 on a done digest and non-zero on a stuck or dead digest, printing the digest JSON in both cases; a refusal keeps its JSON error envelope.
- The recovery prompt names the card and batch gates as allowed and states the tmux-tagged rule.
- A Create of an unexported member in a non-test file, and in a same-package test file, passes the done check.
- create-not-done for a not_found answer names the lookup; for an ambiguous answer it lists each candidate with its file.
- A member declared in `x_linux.go` and `x_other.go` (or `//go:build tmux || llm` and `//go:build !tmux && !llm`) passes a Create done check, and an Edit or Delete of it passes the plan gate.
- Candidates where one file has no constraint, or both match some context, stay ambiguous at the plan gate and the done check.
- delete-not-done blocks while any disjoint candidate remains.
- A Create reusing a name whose every declaration an earlier or same card deletes passes the plan gate; with one declaration left undeleted it still raises create-already-exists.

## Notes for the plan writer

- Shuttle: `internal/shuttleengine/gate.go` (`gateRepromptText`), `spec.go` (`AwaitedShellPrefixes`), `wait.go` (`awaitedLabel`); claudeengine's background-task label.
- Webster: `internal/websterengine/runlevel.go` (`masterAwaitedShellPrefix`, Master spec), `runwrites.go` (`recordedSessions`, `loadRunWrites`), `recoverbatch.go` (uncommitted-path grouping), the state's batch record; `internal/webstercli` for the exit status.
- Stencils: `contracts/stencils/webster/webster-template-master.md`, `webster-prefix-recovery.md`; `contracts/stencils/claims_test.go` pins stencil claims.
- planglyph: `donecheck.go`, `create.go`, `resolve.go` (`candidateList`), `deleteorder.go`, `handle.go`, `drift.go`, `plangate.go`; the status completeness and enforcement tests force every status reader through D7's helper. Check `quarrycli` for other readers of `ResolveResult.Status`.
- Specs: `contracts/specs/loom-plan-spec.md`, `webster-spec.md`, `refusal-spec.md`.

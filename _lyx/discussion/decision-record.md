# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close the Webster faults that stop or mislead runs outside the run-exit audit:
a gate shell cut off when Master's run finishes, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, `recover-batch` exiting 0 on a failed recovery, a recovery prompt that does not name its allowed gates, and Create, Edit and Delete checks that can never pass for a Go member declared once per build-constraint set.

The design faces the code at base 800a0e892; everything from 1e79825ff on is treated as not landed.

## Scope

In:

1. Shuttle holds a turn end on an outstanding `lyx gate test` shell, wherever it sits in the shell's command line, for Master's spawn.
2. The verify gate's re-prompt to Master carries the fixer-fork rule and the fixer prompt path, instead of shuttle's generic "fix every finding".
3. A recovery strand's writes count as the run's own writes, so a later recovery keeps an earlier recovery's uncommitted work as its starting point.
4. `lyx webster recover-batch` exits non-zero on a terminal `stuck` or `dead` digest.
5. The shared implementer body names the card gates, the batch gate, `lyx gate test` with any `--tags` and `lyx loom lint-comments` as always allowed, and says tmux-tagged steps run through `lyx gate test --tags tmux`.
6. `internal/planglyph` reads an ambiguous Go answer whose candidates sit in files with pairwise disjoint build constraints as a resolved build-variant set, at Plan time and at record-batch.
7. Every planglyph finding about a resolve answer that is not found or ambiguous names how the target was looked up and what quarry answered.

Out:

- Changing quarry itself (`github.com/Knatte18/quarry`, a separate module).
- How forks wait for their own gates and the interim notices they cause: that is board entry `fork-wait-noise`.
- Reviving a dead batch through `lyx loom goto` (issue #511's third point).
- Letting a Create handle reuse a name the same card deletes (issue #507's first option).
- Exit codes of `record-batch` and `begin-batch`.

## Decisions

### D1 — A `lyx gate test` shell holds Master's turn end

Decision: Master's spawn declares a second awaited shell prefix, `lyx gate test`, beside `masterAwaitedShellPrefix`.
Shuttle's awaited-label match changes from "the label starts with a prefix" to "some command segment of the label starts with a prefix", where segments are split on `&&`, `||`, `;`, `|` and newlines and trimmed, so `cd <dir> && lyx gate test --tags integration ./x 2>&1 | tail -5` matches.
The match stays generic shuttle code over caller-supplied prefixes; no provider knowledge enters it.
Master's template gains one line: a completion notice for a background shell Master did not start (a fork's gate run) needs no action, so Master ends its turn.

Bound: an awaited gate shell holds a gated Master turn end that would otherwise count Done at once, and nothing else.
It is bounded by the run's own `Timeout` and the liveness check, exactly as the recovery shell and a fork are, and `background_shell_wait_min` stays a display threshold.
A shell matching neither prefix keeps today's rule: it can still be outstanding when the run ends, and `Result.EndedShells`, the `webster-background-shell` friction note and the `RunResult.Warnings` entry record it.
Their wording changes from "lyx did not stop the shell" alone to also saying the shell's result was never read, so no outcome reads as that shell having passed.

Rationale: the failing shells in #505, #509 and #517 were all `lyx gate test --tags integration` runs, and the base already has the awaited-prefix mechanism for exactly this class; the fault is only that gate shells are not in the set and that a `cd … &&` or piped label defeats a pure prefix match.

### D2 — The verify gate's re-prompt carries the fixer-fork rule

Decision: `shuttleengine.GateEntry` gains an optional re-prompt text builder (a function of the findings path returning one line); a nil builder keeps `gateRepromptText`.
Webster's `verify` entry (NewVerifyGate) supplies one that keeps the `Gate findings recorded at <path>` opening Master's template keys on and continues: do not fix them yourself, spawn one fixer fork whose prompt is `<verify-fix prompt path>` exactly as the gate section of your prompt says, and end your turn.
Master's template's "What you never do" gains the gate case explicitly: never edit or commit a source file, a gate failure included.

Bound: the builder replaces only the text of the re-prompt; the send path, `validateSendText`'s one-line rule, the attempt budget and the gate verdict are unchanged.

Rationale: at base the message Master reads at a verify failure says "fix every finding", which contradicts the template's fixer-fork rule at the moment it matters; `driver-context`'s Master followed the message.
A verb that spawns the fixer is not possible, since a fork is an in-session Agent call no CLI can make.

### D3 — Recovery strands' writes are the run's writes

Decision: each recovery spawn records its provider session id on the batch record, in a list carried across respawns and re-begins the way `Recoveries` is.
`loadRunWrites` audits those sessions too and adds their own writes to `RunWrites` as a third group beside Master and Forks.
The recovery prompt's "Written by this run" group therefore holds a path any earlier attempt of the run wrote, an earlier recovery strand included, and the prefix's text says so: the earlier attempt's unfinished work, a fork's or an earlier recovery's, kept as the starting point after checking it against the card.
Reset's own-path set reads the same `RunWrites`, so a reset also treats a recovery strand's writes as the run's own.

Bound: attribution widens only to sessions the run itself spawned and recorded; a path no recorded session wrote stays "Not written by this run".
A recorded session whose transcript cannot be audited is an error naming the session, as for a Master session today.

Rationale: at base `loadRunWrites` reads only Master sessions and their forks, so a recovery strand stopped uncommitted (quick-recipe, #522) leaves 19 paths that the next recovery is told never to touch.
Starting from a clean tree would throw away work the batch's own agent made and force a reset path the run does not otherwise need.

### D4 — `recover-batch` exits non-zero on a failed recovery

Decision: a terminal digest whose status is `stuck` or `dead` returns a failure envelope with exit 1, carrying the same digest fields, `recoveries` and `recovery_retry` it carries today.
A `done` digest and a `--wait` running snapshot keep exit 0.
Master's template rungs keep reading `status` and `recovery_retry` from the envelope; their wording says such a call exits non-zero.

Rationale: a recovery that ended failed and terminal must not read as success to a shell, a script or an operator.

### D5 — The implementer body names the gates that are always allowed

Decision: the shared implementer body (used by forks and the recovery prompt) states, beside its sandbox rule, that the card gates and batch gate it lists, `lyx gate test` with any `--tags` and `lyx loom lint-comments` are not fabric or webster verbs and are always allowed.
A card step or `verify:` line carrying `--tags tmux` or `--tags llm` runs through `lyx gate test` with that flag, like any other step; it is not compile-checked instead.

Rationale: the recovery in quick-recipe hesitated because only bans were named.
`lyx gate test --tags tmux` is permitted from every strand (only raw tagged `go test` is denied), and nothing guarantees the plan-level verify runs the tmux tier, so leaving tmux steps to verify would leave them unrun.

### D6 — A build-variant ambiguous answer reads as resolved

Decision: `internal/planglyph` gets one classifier over a Go `ambiguous` answer: it is a build-variant set when every candidate sits in a distinct file and every pair of those files has disjoint build constraints.
A file's constraint is its `//go:build` expression together with its GOOS and GOARCH filename suffixes, read with `go/build/constraint`; disjointness is decided by evaluating both constraints over every assignment of the tags they mention, with exactly one GOOS and one GOARCH true and `unix` implied by the unix GOOS values.
A file with no constraint overlaps every file; a `_test.go` candidate beside a non-test candidate overlaps it.
Every planglyph reader of a resolve status reads a build-variant set as it reads `multipart`, its candidates as the answer's symbols:
DoneChecks' Create and Rename-new pass, Edit and Uses targets pass `statusFindings`, `answerSymbols` returns the candidates, and Create-already-exists, Delete-not-done and Rename-old keep blocking because declarations still exist.
An ambiguous answer that is not a build-variant set stays `glyph-ambiguous` at Plan time, and its detail names the overlapping pair.

Bound: the classifier turns `ambiguous` into resolved only for a Go glyph whose candidates are provably never compiled together; any unreadable candidate file, unparsable constraint or overlap keeps the answer ambiguous, so the check fails closed.

Rationale: #500 and #507 (`tmuxAllowed`, `StartTime`, `runChild`) and #511 (`rusageStats`, split into `rusage_linux.go` and `rusage_other.go`) are the same fault: idiomatic per-GOOS or per-tag code resolves ambiguous and no glyph check can ever pass.
The check lives in lyx because quarry is a separate module.
Plan time now flags only a genuinely ambiguous Edit, Uses or Delete glyph, and record-batch accepts the build-variant Create the plan asked for.

### D7 — Resolve findings say how the target was looked up

Decision: `create-not-done`, `rename-not-done`, `glyph-not-found` and `glyph-ambiguous` details name the key resolved, that quarry searched the unit's files with test files included, and the answer: `not_found` with whether the unit exists, or `ambiguous` with each candidate's file and, from D6, the overlapping pair.
The webster refusal that carries a `create-not-done` finding keeps its way forward unchanged.

Rationale: #511's operator could not tell a missing symbol from a resolver limit.
The probe at base shows quarry resolves unexported identifiers and test-file declarations, so the true cause was ambiguity, which the message hid.

## Constraints

- `PATTERN-shuttle-provider-seam`: the D1 segment match is generic string handling over caller data in `internal/shuttleengine`; no Claude-specific knowledge moves out of `claudeengine`.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: D1, D2, D3, D4 and D5 change stencil text in `contracts/stencils/webster/`; the D2 re-prompt line is Go-rendered from the verify-fix prompt path Run already holds, and points at the template's gate section rather than restating it.
- `PATTERN-refusal-way-forward`: D4 and D7 change webster refusals' fields and details; each keeps its way forward, and its spec row and test move with it.
- `PATTERN-no-denied-recovery`: D5 names only commands agents' settings allow; `lyx gate test --tags tmux` is allowed, raw tagged `go test` is not named.
- `PATTERN-sole-parsers`: D6 reads build constraints, not plans; plan parsing stays in `planparser`.
- `PATTERN-documentation-lifecycle` and CLAUDE.md's docs rule: `internal/shuttleengine/doc.go`, `internal/websterengine/doc.go`, `internal/planglyph/doc.go` and `internal/webstercli`'s `recover-batch` help update in the same commit as their code; `contracts/specs/loom-plan-spec.md` and `contracts/specs/webster-spec.md` update where they state the changed rules.
- `PATTERN-test-economy`, `PATTERN-test-speed`: extend existing tables (awaited-label, gate re-prompt, run-writes classing, DoneChecks verdicts, statusFindings, stencil claims) before adding tests; unit tier wherever a fake suffices.
- CLAUDE.md test budget: a card adding top-level tests past a package's row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- No new PATTERN entry: no decision adds a cross-cutting invariant.

## Auto-mode assumptions

- A1: the D1 awaited shells in #505, #509 and #517 belonged to Master's session's background tasks, whether Master or a returned fork started them; holding on either is right.
- A2: the recovery strand's provider session id is known to webster at spawn or from the shuttle run record before the strand can write; the plan confirms where it is read.
- A3: quick-recipe's `recover-batch 1` exited 0 on a terminal `dead` or `stuck` digest, the path that returns `output.Ok` at base.
- A4: the tmux tier runs correctly from a recovery strand through `lyx gate test --tags tmux`, as it does from a fork.
- A5: the Go glyph language is the only one with build constraints that matters; other languages keep today's ambiguous handling.

## Open risks

- R1: a long `lyx gate test` shell now keeps Master's run alive up to the run's `Timeout`; a hung gate shell costs that time instead of being killed at strand removal.
- R2: D3 widens reset's own-path set; a reset now also treats a recovery strand's uncommitted writes as the run's to discard.
- R3: D4 changes an exit code a caller may branch on; any sandbox scenario or script reading `recover-batch`'s exit must be updated.
- R4: the D6 tag enumeration is exponential in the tags a candidate pair mentions; real constraints mention few, but the classifier needs a cap past which it fails closed.

## Acceptance criteria

- A gated Master turn end with every output file present and an outstanding shell labelled `lyx gate test …`, `cd <dir> && lyx gate test …` or `lyx gate test … | tail -5` is held, not counted Done; the shell's exit starts the next turn.
- A shell matching no awaited prefix still lets that turn end count Done, and its warning and friction note say its result was never read.
- An awaited gate shell still ends at the run's `Timeout`.
- The verify gate's re-prompt to Master starts with `Gate findings recorded at`, names the verify-fix prompt path, and tells Master not to fix the findings itself; every other gated run keeps the generic text.
- A second recovery spawned after a first recovery stopped uncommitted lists the first recovery's written paths under "Written by this run".
- A path no recorded session wrote stays under "Not written by this run".
- `recover-batch` on a terminal `stuck` or `dead` digest exits non-zero and its envelope carries `status`, `recoveries` and `recovery_retry`; on `done` and on a `--wait` snapshot it exits 0.
- The rendered recovery prompt and fork prompt name `lyx gate test` with any `--tags` and `lyx loom lint-comments` as always allowed.
- An unexported Create target and a Create target declared in a `_test.go` file pass DoneChecks once declared.
- A Create target declared once in a `//go:build tmux || llm` file and once in a `//go:build !tmux && !llm` file passes DoneChecks; one declared in `_linux.go` and `_windows.go` passes too.
- An Edit target with per-GOOS bodies passes the plan validator; a Delete or Rename-old of such a target blocks until every variant is gone.
- Two candidates with overlapping constraints, or one without a constraint, stay `glyph-ambiguous` at Plan time and `create-not-done` at record-batch, and the detail names the overlapping files.
- A `create-not-done` detail names the resolved key, the answer status and, for `not_found`, whether the unit exists.

## Notes for the plan writer

- D1: `masterAwaitedShellPrefix` and its use in `internal/websterengine/runlevel.go`; `awaitedLabel` in `internal/shuttleengine/wait.go`; the corpus case `master-fork-running` already models a shell left by a returned fork.
- D2: `gateRepromptText` in `internal/shuttleengine/gate.go`; `NewVerifyGate` in websterengine; `contracts/stencils/claims_test.go` pins the `Gate findings recorded at …` claim.
- D3: `loadRunWrites` and `recordedSessions` in `internal/websterengine/runwrites.go`; `BatchState` in `state.go`; the `uncommitted_paths` grouping in `recoverbatch.go`; `webster-prefix-recovery.md`.
- D4: the terminal `output.Ok` in `internal/webstercli/recoverbatch.go` and its `Long` help.
- D6 and D7: `doneCheckVerdicts` in `internal/planglyph/donecheck.go`, `statusFindings` in `resolve.go`, `createFindings` in `create.go`, `answerSymbols` in `deleteorder.go`.

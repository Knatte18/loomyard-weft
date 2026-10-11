# Decision record — twb-so-oh-1: Webster stability (gate shells, Master self-fix, recovery, Create checks)

## Goal

Close the Webster faults that stop or mislead runs outside the audit:
a background `lyx gate test` shell cut off at Master's turn end so a run reads `done` over an unfinished gate tier;
Master fixing a verify failure itself because the gate re-prompt tells it to;
a second recovery treating the first recovery's uncommitted work as foreign, and `recover-batch` exiting 0 over a failed batch;
a recovery prompt that leaves its allowed gates unclear;
and Create, Edit and Delete checks that can never pass for a member declared once per build-constraint set or per GOOS.

The record faces the code at base `800a0e892`; nothing from `1e79825ff` on is assumed landed.

## Scope

In:

1. Master's wait holds its turn end on an outstanding `lyx gate test` shell, and a gate shell the run still ends over is recorded as an unread result.
2. The verify gate's re-prompt to Master carries webster's own instruction (spawn one fixer fork), replacing shuttle's generic "fix every finding".
3. Recovery write attribution covers every earlier recovery strand of the run; `recover-batch` exits non-zero on every terminal digest but `done`.
4. The implementer body and recovery prefix name the card and batch gates as always allowed, and say the `tmux` tier is compiled, never run.
5. The `create-not-done` refusal names the resolve status and candidates, so a missing symbol is told apart from a resolver answer webster does not accept.
6. planglyph reads an ambiguous answer whose candidates are build variants of one member (same unit, pairwise disjoint build constraints) as multipart, at every status read: plan gate, Create, Edit, Delete, Rename, DoneChecks and delete ordering.

Out:

- Making quarry evaluate build constraints (quarry is a separate module and documents that it does not, on purpose).
- Letting a Create handle reuse a name its own card deletes (#507's first option): item 6 makes per-GOOS Edit and Delete by glyph work, which removes the need.
- Reviving a dead batch through `lyx loom goto` (#511 point 3).
- How forks run gates and quick checks, and fork interim notices: owned by board entry `fork-wait-noise`.
- Stopping a background shell from lyx: the provider reports no pid.

## Decisions

### D1 — Master awaits `lyx gate test` shells

Decision: webster's Master spawn declares `lyx gate test` as a second awaited shell prefix beside `masterAwaitedShellPrefix` (`lyx webster recover-batch`).
Shuttle's awaited-shell match changes from "the label starts with a prefix" to "some command segment of the label starts with a prefix", segments being the label split at `&&`, `||`, `;`, `|` and newlines and trimmed, so `cd <pair> && lyx gate test --tags integration ./x 2>&1 | tail -5` matches.
The splitter is a pure, stdlib-only function in `internal/shell`, which already owns shell syntax; shuttle's matcher calls it.
`Spec.AwaitedShellPrefixes`' doc comment and shuttle's `doc.go` state the segment rule.
Rationale: at base no shell expires any more, but a gated run's waiting turn end with every output file present still finishes Done at once when every outstanding task is an unawaited shell, and strand removal then kills the gate.
Holding the turn end lets the gate finish and its completion reach Master before the run can read `done`.
Bound: the hold adds no new expiry and skips nothing; an awaited shell is bounded only by Master's run `Timeout` (`master_timeout_min`) and the liveness check, exactly as the recover-batch shell is today.
A false-positive segment match (operator text inside quotes) only holds a turn end longer, under the same bound.

### D2 — A gate shell the run ends over reads as an unread result

Decision: when `Result.EndedShells` holds a shell whose label has a `lyx gate test` segment, webster's per-shell warning, friction note and summary line say the gate run was unfinished and its result was never read, so the run did not pass it.
The match uses the same segment rule as D1, through one webster helper.
Other shells keep today's wording.
Rationale: after D1 a gate shell can still be outstanding at a run's end on timeout, died, or a Master that wrote its outputs and was then stopped; the issue's floor is that such an outcome never reads as "tests passed".
lyx still stops no shell: the provider reports no pid, so "stopped explicitly" is not available.

### D3 — The gate entry carries its own re-prompt instruction

Decision: `shuttleengine.GateEntry` gains an optional `Reprompt` string: the instruction clause shuttle sends after `Gate findings recorded at <path> — ` when that entry is the first failing entry (`gateFailedAt`).
Empty keeps today's default clause, `read it, fix every finding, and end your turn.`
Webster's verify gate entry sets it to a single line telling Master not to edit, commit or fix anything itself, to spawn exactly one fixer fork whose prompt file is the rendered verify-fix prompt (its path in the line), as its "A gate failure: spawn one fixer fork" section says, and to end its turn.
The Master stencil's section keeps its steps and is checked to agree with the line.
Rationale: the incident's root cause is shuttle's generic re-prompt, which literally tells Master to "fix every finding" at the moment Master reads the failure, contradicting its stencil.
Stating the rule where the failure is read is the issue's first expected fix; a verb cannot spawn an in-session fork, so the second is not available.
Bound: the field changes only the text sent; it spends, skips and adds no attempt, gate evaluation or send.

### D4 — Earlier recoveries are the run's own writes

Decision: `BatchState` records the session id of every recovery spawn of the batch in a list carried across respawns and re-begins, the way `Recoveries` is; `reset --to batch-start` clears it with the count, and `reset --to start` archives it with the record.
`loadRunWrites` audits those sessions too and returns their writes in a third `RunWrites` field of their own, so the contract-file evidence, which reads Master's writes, is unchanged.
The recovery prompt's uncommitted-path grouping and the reset own-path set count recovery writes as written by the run.
The "Written by this run" guidance in `webster-prefix-recovery.md` says the group includes an earlier recovery's unfinished work.
An audit error on a recovery session is returned like a Master session's, so `recover-batch` refuses with a way forward instead of rendering a misattributed block.
Rationale: attribution by batch, as the issue expects: write evidence today covers Master and fork sessions only, so a stopped first recovery's 19 files landed under "Not written by this run" and blocked the second.
Keeping the tree and grouping it correctly preserves the work; a clean-tree restart would need a destructive reset outside the recovery's reach.

### D5 — `recover-batch` exits non-zero on a terminal non-done digest

Decision: a terminal digest whose status is not `done` (stuck, dead, failed) exits non-zero through an error envelope that carries every field the success envelope carries today (`batch`, `status`, `dead_reason`, `recoveries`, `recovery_retry`, `warnings` and the rest of the digest fields).
Its message names the status and the way forward: `lyx webster recover-batch NN` once more when `recovery_retry` is true, otherwise the run ends stuck and the operator's route is `lyx webster reset --to batch-start` or `--to start`.
A running snapshot and a done digest still exit 0.
The Master stencil's recover-batch rungs say the non-done digests arrive as a non-zero exit with the same fields, and `refusal-spec.md` gains the row, per `PATTERN-refusal-way-forward`.
Rationale: a batch that ended failed and terminal must not read as success to a caller that checks the exit code.
Bound: only the exit code and envelope wrapper change; the rungs' fields and their meaning do not.

### D6 — The implementer prompt names its always-allowed gates

Decision: the shared implementer body's sandbox section states that the card gate steps, the batch gate steps and a card's `verify:` lines (`lyx gate test`, `lyx loom lint-comments`, package-scoped `go` runs) are always the implementer's to run, and are neither `lyx webster` nor `lyx fabric` verbs.
Its raw-`go` section states that a `tmux`-tier check is compiled with `go vet -tags tmux <packages>` and never run: Publish runs that tier.
The recovery prefix's `lyx webster` ban says it covers `lyx webster` only and points at the sandbox section for what the strand always runs.
Rationale: the recovery strand could not tell whether its gates fell under the bans, and both bans live in the text it reads; stating it in the shared body says it once and serves forks too.

### D7 — `create-not-done` says how the target was looked up

Decision: the `create-not-done` detail, and the `rename-not-done` new-side detail, name the resolve key looked up, quarry's status, and for `ambiguous` the candidates with their files plus why they were not accepted as build variants (a file with no constraint, overlapping constraints, or different units).
Unexported identifiers and identifiers in `_test.go` files resolve through quarry today; an acceptance row pins both.
Rationale: both #511 targets were members declared once per build-constraint set, so the failure was an `ambiguous` answer the check rejected, not an unexported-name limit, and the message could not show that.

### D8 — Build variants of one member read as multipart in planglyph

Decision: planglyph normalizes each quarry answer at its one batched-resolve chokepoint: an `ambiguous` answer becomes `multipart`, its candidates moved to its symbols, when every candidate sits in the same unit directory and the candidates' files carry pairwise disjoint build constraints.
A file's constraint is its `//go:build` line (parsed with `go/build/constraint`) conjoined with its filename's `_GOOS`, `_GOARCH` or `_GOOS_GOARCH` suffix.
Two constraints are disjoint when no tag assignment satisfies both, where exactly one known GOOS and one known GOARCH hold (with `unix` implied by the unix GOOS values) and every other tag is free.
The known GOOS and GOARCH lists are declared once in the helper.
A file without a constraint overlaps every file, so a plain duplicate stays `ambiguous`; so does a unit collision.
Every planglyph status read sees the normalized answer: the plan gate's status policy, `create-already-exists`, DoneChecks' Create, Delete and Rename rules, delete ordering's `answerSymbols`, handle binding and drift.
The plan gate therefore accepts such a glyph at Plan time exactly when record-batch will, and a genuinely ambiguous glyph is still flagged at Plan time as `glyph-ambiguous`, its detail naming why the candidates are not build variants (shared with D7).
Rationale: quarry's `multipart` already means "one member the language lets be declared in several places"; a member declared once per build-constraint set is that case.
Normalizing once, in planglyph, keeps quarry's documented choice and keeps every check agreeing.
Bound: the normalization lets through only candidate sets proven pairwise unsatisfiable together; an expression it cannot parse, or a tag set too large to enumerate within a fixed cap, counts as overlapping and the answer stays `ambiguous`, so it fails closed.
Delete keeps its rule: any declaration still present blocks.

## Constraints

- `PATTERN-shell-mechanics-seam`: `internal/shell` stays stdlib-only; the D1 splitter is a pure function there and builds no command string.
- `PATTERN-shuttle-provider-seam`: the segment match is provider-neutral and lives in shuttle proper; label extraction stays in `claudeengine`.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D5's non-zero exit, D4's audit refusal and D7's detail each name a way forward an agent's settings allow, with `refusal-spec.md` rows and tests.
- `PATTERN-stencil-ownership`, `PATTERN-producer-pointer-rule`: stencil text changes go through the existing stencils, read at call time; D6 points at sections rather than restating them.
- `PATTERN-sole-parsers`: no new plan, discussion or summary parser; D8 reads Go build constraints only, through the standard library.
- `PATTERN-glyph-conversion-chokepoint`: D8 reads candidate file paths from quarry's answer and converts no glyph to a path itself.
- `PATTERN-test-speed`, `PATTERN-test-economy`: unit tier with fakes for shuttle and webster; planglyph's existing real-quarry fixtures for D8; extend existing tables before adding top-level tests.
- `PATTERN-comment-line-breaks` and the docs rule: `doc.go` of `shuttleengine`, `websterengine` and `planglyph`, and `recover-batch`'s help text, change in the same commit.
- Test budget: a card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- No new PATTERN entry: each rule here is package-internal and lives in its package's `doc.go`.

## Auto-mode assumptions

- A finished background shell reaches Master as a new turn (Claude Code's completion notification), including a shell a fork launched and left behind, so D1's hold releases into an ordinary turn end. See Open risks.
- Master's spawn can learn a recovery strand's session id at spawn time the same way `runlevel.go` learns Master's (`shuttleengine.FindRun`).
- The verify gate is the only Master gate entry that needs a non-default re-prompt; entries in `deps.Gate` keep the default.
- Fork gate behavior (waiting in-turn, foreground quick checks) is left to `fork-wait-noise`; D1 does not depend on it and does not contradict it.
- "Left to verify" for the `tmux` tier means Publish's `publish_verify`, since a plan's `## verify:` only compiles that tier (`loom-plan-spec.md`).

## Open risks

- If a fork's leftover shell completes without producing a Master turn, D1's hold lasts until `master_timeout_min`. Mitigation: the acceptance criteria require the release path to be covered; if Plan finds no turn arrives, it re-reads the waiting list from the engine on poll ticks rather than adding an expiry.
- D8's GOOS and GOARCH lists can drift from the toolchain's; a missing value makes a pair read as overlapping, which fails closed.
- D5 changes an exit code agents already read; a stencil rung that keys on "completes" rather than the fields could misroute until the Master stencil is updated in the same commit.

## Acceptance criteria

- A gated Master run whose turn end has every output file present and an outstanding shell labelled `cd <path> && lyx gate test --tags integration ./x 2>&1 | tail -5` keeps waiting; the next turn end with nothing outstanding arrives at the gate.
- The same with only an unrelated unawaited shell still finishes Done at once, as today.
- An awaited gate shell is bounded by the run's `Timeout`, and a run that times out over it records the shell in `Result.EndedShells`.
- The segment splitter splits at `&&`, `||`, `;`, `|` and newlines and trims each segment; a prefix inside a non-leading segment matches, a prefix only inside a `grep` argument does not.
- A webster run ending over a `lyx gate test` shell writes a warning, friction note and (on done) summary line saying the gate run was unfinished and its result never read; a non-gate shell keeps today's wording.
- A verify-gate failure sends Master a re-prompt naming the findings path, forbidding a self-fix and naming the fixer fork prompt path; a gate entry without `Reprompt` sends today's default text.
- With a stopped first recovery's uncommitted files in the tree, the second recovery's prompt lists them under "Written by this run"; a file no session of the run wrote stays under "Not written by this run".
- Contract-file evidence and the run-exit parent-write audit ignore recovery sessions' writes.
- `reset --to batch-start` clears the batch's recorded recovery sessions.
- `recover-batch` exits non-zero with every digest field for a terminal stuck, dead or failed digest, with a way forward naming the retry or the reset route; it exits 0 for done and for a running snapshot.
- The rendered recovery prompt states the card and batch gates as always allowed and the `tmux` tier as compiled with `go vet -tags tmux`, never run; the stencil claims tests pin both.
- A Create of an unexported member, and of a member declared in a `_test.go` file, passes DoneChecks.
- A Create whose member is declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`) passes DoneChecks; so does one declared in `x_linux.go` and `x_other.go` (`//go:build !linux`).
- An Edit and a Delete of a per-GOOS member pass the plan gate, and the Delete's done-check blocks while any variant remains.
- Two declarations in unconstrained files, overlapping constraints (`linux` and `unix`), or a unit collision stay `ambiguous`: the plan gate reports `glyph-ambiguous` and DoneChecks `create-not-done`, each detail naming the status, the candidates with files, and why they are not build variants.
- A constraint expression the helper cannot parse leaves the answer `ambiguous`.

## Notes for the plan writer

- Shuttle: `awaitedLabel` and `expiredTurnEnd` in `internal/shuttleengine/wait.go`; `gateRepromptText` in `gate.go` and its one call in `handleGatedBoundary`.
- Webster: `masterAwaitedShellPrefix` and the spawn spec in `internal/websterengine/runlevel.go`; `expiredShellWarning`, `writeBackgroundShellFrictionNote` and `AppendBackgroundShells` for D2; `NewVerifyGate`'s entry for D3.
- Recovery: `runwrites.go` (`loadRunWrites`, `recordedSessions`), `uncommittedPathsBlock` in `recoverbatch.go`, `BatchState` in `state.go`; the CLI exit in `internal/webstercli/recoverbatch.go`.
- planglyph: `resolveTargets` is the batched-resolve call DoneChecks uses; check every status read (`resolve.go`, `create.go`, `donecheck.go`, `deleteorder.go`, `handle.go`, `drift.go`, `plangate.go`) reads through the normalization.
- quarry v0.2.1's `statusForMatches` documents why it ignores build constraints.
- Stencils: `contracts/stencils/webster/webster-template-master.md`, `webster-body-implementer.md`, `webster-prefix-recovery.md`; claims in `contracts/stencils/claims_test.go`.

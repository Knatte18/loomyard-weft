# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close six Webster faults that stop runs or make them read wrong: a gate shell cut off at Master's turn end, Master fixing a verify failure itself, a second recovery treating the first recovery's files as foreign, a recovery prompt that does not name its allowed gates, a `create-not-done` refusal that does not say how it looked the target up, and build-constrained Go members that resolve `ambiguous` and so can never pass a Create or Edit check.

## Scope

In:

- Master's shuttle spawn spec in `internal/websterengine`: `lyx gate` shells join the awaited shell prefixes.
- A per-entry re-prompt text on shuttle's `GateEntry`, used by Webster's verify gate.
- The Master stencil's verify-failure and gate-shell rules.
- Recovery write attribution by batch: each recovery strand's session joins the run's write evidence.
- The exit code of `lyx webster recover-batch` on a terminal-failed digest.
- The recovery prompt naming its allowed gate commands and the tmux-tier rule, in the shared implementer body.
- The `create-not-done` and `rename-not-done` details naming the lookup and its answer.
- A Go build-constraint rule in `internal/planglyph` that reads an `ambiguous` answer whose candidates sit in files with pairwise disjoint build constraints as one declaration, at Plan time and at record-batch alike.
- The docs this changes: `internal/websterengine/doc.go`, `internal/shuttleengine/doc.go`, `internal/planglyph/doc.go`, and the plan spec rows for the changed check details.

Out:

- Changes to quarry itself (`github.com/Knatte18/quarry`, a separate module).
- Reviving a dead batch through `lyx loom goto` (raised in #511, not part of this entry).
- Letting a `Create` reuse a name the same card deletes (#507's first option); D9 makes such a symbol editable in place, which removes the need.
- Process-lifetime consolidation (`proc-child-lifetime`) and gate hygiene items (`gate-test-hygiene`).

## Decisions

### D1. A `lyx gate` shell holds Master's turn end

Decision: Master's spawn spec adds `lyx gate ` to `AwaitedShellPrefixes`, beside the existing `lyx webster recover-batch` prefix.
A turn end with Master's output files present and a `lyx gate` shell outstanding keeps waiting, exactly as for a fork.
Bound: the awaited shell holds the turn only until it exits or Master's run deadline (`master_timeout_min`) or the liveness check ends the run; it never extends that deadline.
A shell whose command does not start with `lyx gate ` (for example one prefixed with `cd … &&`) stays unawaited, as today.

Rationale: shuttle already has the mechanism and the code path that ended #505/#509/#517 runs is `countsAtOnce`, which counts a fresh gated run's turn end at once when only unawaited shells remain.
Making gate shells awaited closes that path without a new rule in shuttle.

### D2. A gate shell a run ends over is recorded as discarded

Decision: the warning Webster writes for each `Result.EndedShells` entry states, for a shell whose label starts with `lyx gate `, that the gate's result was never read and counts as no result.
The existing recording (run warnings, `AppendBackgroundShells`, the friction note, summary) stays.

Rationale: after D1 such a shell can only end with a non-done outcome or an ungated path, and the board asks that the outcome never read as "tests passed".

### D3. The Master stencil tells Master not to start gate runs itself

Decision: the Master stencil states that Master never runs `lyx gate` or any test command; the verify gate runs verify after Master's turn ends, and forks run card and batch gates.
D1 still holds for a gate shell Master starts against that rule.

Rationale: Master's gate runs duplicate the verify gate and were the shells cut off; the rule removes the cause, D1 makes a breach safe.

### D4. Webster's verify-gate re-prompt names the fixer fork

Decision: `GateEntry` gains an optional re-prompt text builder taking the findings path; empty keeps today's `gateRepromptText`.
Webster's verify-gate entry supplies one line that reads, in substance: verify findings are recorded at the path; do not edit or commit; spawn one fixer fork with the prompt file at `verify_fix_prompt_path`, end the turn, and on its notification rewrite the outcome and summary.
The Master stencil's `## A gate failure` section keys on the new text's opening words, and the shuttle and webster docs record that an entry may set its own re-prompt.
Bound: the builder changes only the sent text; attempt counting, findings files, `Terminal` and `PassOnCap` behave as before, and the text must stay one line (`validateSendText`).

Rationale: in `driver-context` Master obeyed the message it was sent, `Gate findings recorded at … fix every finding`, which contradicts its stencil.
Stating the rule in the message Master reads at the failure makes the fixer fork the easy path.
A verb that spawns the fixer is not possible: a fork is an in-session Agent call, not a process a verb can start.

### D5. Recovery writes count as the batch's own work

Decision: the batch state records each recovery strand's session id, appended per spawn and never overwritten, and `recordedSessions` includes them, so `loadRunWrites` attributes a path any earlier recovery of the run wrote to "Written by this run".
A recovery session is audited like a Master session: its parent write events are its writes.
A path no session of the run wrote, Master, fork or recovery, stays under "Not written by this run".

Rationale: attribution is already by run, not by shuttle run id; the gap is that recovery strands, being separate sessions, were never in the session list.
Keeping the first attempt's work in place matches the recovery prompt's existing "keep it as the starting point" rule and avoids a destructive reset of uncommitted work.

### D6. `recover-batch` exits non-zero on a terminal-failed batch

Decision: `lyx webster recover-batch` exits 1 when its terminal digest leaves the batch terminal-failed: `stuck`, or `dead` with `recovery_retry: false`.
It exits 0 on `done` and on `dead` with `recovery_retry: true`.
The JSON body is unchanged, so Master's rungs read it as before.

Rationale: an exit 0 on a batch that ended failed reads as success to anything that checks the exit code, including the operator and the background-shell notification.

### D7. The implementer body names the always-allowed gate commands

Decision: the shared `webster-body-implementer` stencil, which the recovery prompt also renders, states that every card gate step, the batch gate, `lyx gate test` at any package scope and `lyx loom lint-comments` are always allowed, and that the fabric ban covers mutating `lyx fabric` verbs only, as the recovery prefix's ban covers `lyx webster` only.
It states that a `tmux`-tagged step is compile-checked (`go vet -tags tmux`) and its run is left to Publish's verify, matching the plan stencil's rule.
The stencil claims test pins both statements.

Rationale: the recovery strand in `quick-recipe` was unsure whether gates fell under the ban; the rule is the plan stencil's already, so this adds no new skip.

### D8. Done-check details say how the target was looked up

Decision: `create-not-done` and the new-side `rename-not-done` details name the glyph resolved, quarry's status, and for `ambiguous` the candidate files and that their build constraints overlap or none was found; for `not_found` they say the lookup covered every file of the unit, test files included.
The matching rows in `contracts/specs/loom-plan-spec.md` change in the same commit.

Rationale: #511 asked whether unexported or test-file identifiers resolve; they do (both resolve `found`, an external test package as `<dir>_test#Name`), and the two batches died on build-constraint ambiguity, which the old detail hid.

### D9. A Go symbol declared once per disjoint build configuration resolves as one declaration

Decision: `internal/planglyph` gains one predicate over a quarry `ambiguous` answer: it is one declaration when every candidate is a `.go` file and no build configuration selects two candidate files.
A file's constraint is its `//go:build` line together with its filename's GOOS/GOARCH suffix, parsed with `go/build/constraint`; two files are disjoint when no assignment satisfies both, with GOOS and GOARCH each taking exactly one value.
Every planglyph site that reads a resolve answer for an existing symbol uses it: the Create and Rename-new done checks count it as resolved, the Plan-time Edit, Delete and Uses checks accept it, and `create-already-exists` treats it as existing.
The Delete and Rename-old done checks keep requiring `not_found`.
A file that cannot be read or whose constraint does not parse makes the predicate false, so the answer stays ambiguous and blocks as today.
Bound: the predicate lets through only an ambiguous answer whose every candidate pair is provably disjoint; overlapping or unconstrained candidates still block, and non-Go answers are untouched.

Rationale: quarry is a separate module and has no build-context input; lyx owns the policy over its answers, and one predicate used at Plan time and at record-batch keeps the two in step, which is what the board asks for "flag at Plan time rather than at record-batch".

## Constraints

- `PATTERN-shuttle-provider-seam`: D1 and D4 are caller data on the spec and the gate entry; nothing provider-specific enters `shuttleengine` or `reedengine`.
- `PATTERN-refusal-way-forward`: D6 changes no refusal text; D8's changed details land with their spec rows and a test reaching each.
- `PATTERN-no-denied-recovery`: D4's re-prompt and D7's allowed list name only commands agents may run.
- `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-stencil-ownership`: stencil edits stay in `contracts/stencils/webster/`, read through `stencilstore`.
- `PATTERN-sole-parsers`: D9 reads Go source build lines, not plan or discussion files; plan parsing stays with `planparser`.
- `PATTERN-ref-shape-registry`: D9 changes no ref-shape classification.
- `PATTERN-test-speed`, `PATTERN-test-economy`: tests sit in the lowest tier their substrate needs and extend existing tables where one covers the surface.
- `cmd/lyx/testdata/test-budget.yaml`: a card whose new top-level tests push a package past its row lists that file under `Edit:`.
- Docs land in the same commit as their code (`CLAUDE.md`).

## Auto-mode assumptions

- No operator answered; every decision above is an auto-pick logged in the support log's question ledger.
- The `lyx gate ` prefix match on the shell's command label is the right granularity; a gate run wrapped by `cd` or another prefix is rare enough to leave unawaited.
- Exit 1 on a terminal-failed `recover-batch` digest does not break Master's flow, since Master reads the JSON body from the completion notification whatever the exit code.

## Open risks

- D1 makes a long `lyx gate test --tags integration` Master started hold the run up to `master_timeout_min`; D3 is what keeps that rare.
- D9's GOOS and GOARCH value lists must cover every value used in this repo's filenames and build lines; an unknown value makes the predicate fail closed, which reproduces today's block, not a false pass.
- D5 relies on the recovery session's transcript being readable by `AuditForks` after the strand is stopped; if a stopped strand's transcript is gone, its paths fall back to "Not written by this run".

## Acceptance criteria

- A gated Master run, started fresh, whose turn ends with its output files present and a `lyx gate test …` shell outstanding keeps waiting until the shell ends; a shell with another label counts at once as today.
- A run that ends with a `lyx gate` shell outstanding records a warning saying its result was never read.
- The verify gate's re-prompt to Master is the Webster text naming the fixer prompt path; another gated producer still gets the default text.
- A re-prompt builder returning multi-line text is refused the way any multi-line send is.
- A second recovery's prompt lists paths the first recovery wrote under "Written by this run", and a path no session wrote under "Not written by this run".
- `recover-batch` exits 1 on `stuck` and on `dead` without a retry, and 0 on `done` and on `dead` with a retry, with an unchanged JSON body.
- The rendered recovery prompt names `lyx gate test` and `lyx loom lint-comments` as allowed and states the tmux-tier rule.
- `create-not-done` for an ambiguous target names its candidate files; for a not-found target it says test files were searched.
- A member declared in two files with disjoint `//go:build` lines, and one declared per GOOS by filename suffix, passes the Create done check and a Plan-time Edit check.
- A member declared in two files whose constraints overlap, or where one file is unconstrained, still fails both.
- A Delete target answering disjoint-ambiguous still fails `delete-not-done`.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass.

## Notes for the plan writer

- Turn-end logic: `countsAtOnce` and `expiredTurnEnd` in `internal/shuttleengine/wait.go`; Master's prefix constant is `masterAwaitedShellPrefix` in `internal/websterengine/runlevel.go`.
- The default re-prompt is `gateRepromptText` in `internal/shuttleengine/gate.go`; the Master stencil's gate-failure section is in `contracts/stencils/webster/webster-template-master.md`.
- Write attribution: `uncommittedPathsBlock` in `recoverbatch.go`, `loadRunWrites` and `recordedSessions` in `runwrites.go`.
- Done checks: `doneCheckVerdicts` in `internal/planglyph/donecheck.go`; probe with `lyx quarry resolve internal/proc#StartTime` (per-GOOS, ambiguous) and `internal/testkit/tmuxkit#tmuxAllowed`.

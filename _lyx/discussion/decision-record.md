# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close the Webster faults outside the run-exit audit that stop a run or let it report a misleading outcome, as the board entry's six parts list them.
The record faces bench base `800a0e892`; every premise below was read there.

## Scope

In:

1. Background gate shells (#505, #509, #517): Master's gated wait holds the turn end on an outstanding gate or recovery shell, matched by command shape; Master runs no gate itself; the record of a shell ended with the run says its result was never read.
2. Master self-fix: the verify gate's re-prompt and findings file tell Master to spawn the fixer fork and edit nothing.
3. Recovery attribution and exit code (#522): recovery strands' sessions join the run's write evidence; `recover-batch` exits non-zero whenever its answer is a terminal batch that is not `done`.
4. Recovery gates: the shared implementer body names the gates always allowed; the recovery prompt says a `tmux`- or `llm`-tagged test is compiled, not run.
5. Create check (#511): `create-not-done`, `delete-not-done` and `rename-not-done` details say how the target was looked up and what quarry answered.
6. Partitioned members (#500, #507): a glyph declared once per mutually exclusive build configuration counts as one member declared in several places at the plan-side resolve sites and in handle binding.

Out:

- A change to quarry (`github.com/Knatte18/quarry` v0.2.1, an external module); quarry keeps answering `ambiguous` and lyx classifies.
- lyx killing a background shell: the provider reports no pid, and lyx stops no shell today.
- How a fork launches and waits on its gates, and the wake-per-gate-wait noise: `fork-wait-noise`'s.
- #507's "a Create handle may reuse a name the same card deletes": with part 6 a partitioned member is edited by glyph.
- A Rename whose old side is a partitioned member: `renameDeclSource` accepts `found` only, and it stays `rename-old-unresolved`.
- #511's third point, reviving a dead batch through `lyx loom goto`, which the board entry does not ask for.
- A resolver change for unexported or same-package test-file members: quarry v0.2.1 already resolves them.

## Decisions

### D1 — A gate or recovery shell holds Master's turn end

**Decision:** `shuttleengine.Spec` gains a command-shape match beside `AwaitedShellPrefixes`: a shell's label (its command text) counts as awaited when any of its shell segments, split on `&&`, `||`, `;` and `|`, starts with a declared prefix after leading `NAME=value` words.
The splitter lives in `shuttleengine` or a package below it.
Webster declares two prefixes for Master: `lyx webster recover-batch` (today's) and `lyx gate`.
A matched shell holds Master's turn end as a fork does today: `countsAtOnce` never counts it, and the turn end waits for the next event.
The hold is bounded by what bounds every held turn end: the run's deadline (`master_timeout_min`) and the liveness check.
A deadline or liveness end with the output files present still evaluates the gate (`evaluateGate(final)`, which runs the verify entry's own `Gate` since it sets no `Final`), and `finishMasterDone` already demotes a `done` whose gate did not pass to `stuck`.
A shell that matches no prefix keeps today's handling.
For every shell outstanding when the run ends, the existing `RunResult.Warnings` entry, `webster-background-shell` friction note and summary section say its result was never read.
**Rationale:** `countsAtOnce` finishes a gated run `done` at once when every outstanding task is an unawaited shell and the output files exist, which is the reported case; `awaitedLabel` is a whole-label prefix, so `cd X && lyx gate test …` or `lyx gate … | tail` never matched.
A run that ends `done` has passed the verify gate, which runs the plan's own `## verify:`, so an ended shell's lost result never stands in for the run's tests; the wording says so plainly.

### D2 — Master runs no gate itself

**Decision:** the Master template states, beside its rule that Master never runs the plan-level verify, that Master runs no `lyx gate` command.
**Rationale:** the incident shells were Master's own off-script runs; Master drives bracket verbs and judges digests, and D1 stays the backstop for a lapse or a shell a fork left behind.

### D3 — The verify re-prompt names the fixer fork

**Decision:** `shuttleengine.GateEntry` gains an optional one-line re-prompt text, rendered from the findings path; an entry without it keeps `gateRepromptText`'s generic line.
Webster's `verify` entry sets a line that names the findings file, says not to edit or commit, and says to spawn one fixer fork on the verify-fix prompt file and end the turn.
`renderVerifyGateFindings` opens the findings file with the same instruction.
The text names no command an agent's settings deny.
**Rationale:** the generic line, "read it, fix every finding, and end your turn", is what Master reads at the moment it acts, and it contradicts the template's fixer-fork section.

### D4 — A recovery strand's writes are the run's own

**Decision:** each recovery spawn records its Claude session id on the batch record, in a list that carries across re-begins and respawns as `Recoveries` does, inside the state commit `recover-batch` already makes right after a spawn.
A call that attaches to a live strand records nothing new.
`loadRunWrites` audits those sessions into a separate `RunWrites` slice, and the recovery prompt's "Written by this run" grouping and a reset's own tracked paths take it.
The contract-file check (`contractevidence.go`) keeps reading Master and fork writes only.
A recorded recovery session with no transcript on disk is skipped with a warning, and its paths stay unattributed, the side the prompt leaves untouched.
A state written before the field existed reads as no recovery sessions.
**Rationale:** `recoverSpawn` already reads the strand's `RunState.SessionID` but records none, so a first recovery's edits fall under "Not written by this run".
Attribution through evidence keeps the earlier attempt's work as the next attempt's starting point, which the recovery prompt already promises; the skip only widens the untouched side.

### D5 — `recover-batch` exits non-zero on a terminal batch that is not `done`

**Decision:** every `recover-batch` call whose answer is a terminal batch with a status other than `done` returns `ok: false` and exits 1, carrying the digest fields (`batch`, `status`, `head_sha`, `deviations`, `dead_reason`, `elapsed_s`, `recoveries`, `recovery_retry`, `warnings`).
At base that changes the terminal `stuck` and `dead` digests, which return `output.Ok`; `failed` already exits 1.
Its `error` names the way forward: for `dead` with `recovery_retry: true`, run `recover-batch NN` once more; otherwise the run ends stuck naming the batch.
`done` and the non-terminal `running` snapshot stay exit 0.
Master's ladder rungs read the same fields off the failure envelope.
The existing refusal-note writer (`noteRefusals`) records each such envelope as a `webster-refusal-recover-batch` note; no second note is added.
**Rationale:** a recovery that ended stuck or dead is a failed command; with exit 0 an operator or a script reads it as a success.

### D6 — The implementer body names the gates always allowed; the recovery prompt handles tagged tests

**Decision:** `webster-body-implementer.md`, shared by fork and recovery, says next to its sandbox rule that the card gate and batch gate steps, `lyx gate test` over the card's packages, `lyx loom lint-comments` and `lyx quarry resolve` are always allowed, and that the fabric ban covers mutating `lyx fabric` verbs only.
It says what may run, never how a call is launched, which `fork-wait-noise` owns.
`webster-prefix-recovery.md` adds that a test in a `tmux`- or `llm`-tagged file of the card's packages is compiled, not run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, and points at the plan template's tier paragraph for where that tier runs (Publish's `publish_verify`).
**Rationale:** nothing denies these commands; the strand's doubt came from a text that names only bans.
The tagged-test rule goes in the recovery prefix only, as the board asks, so forks keep today's behaviour; the plan template already owns the tier contract, so the prefix points at it.

### D7 — Done-check details say how the target was looked up

**Decision:** the `create-not-done`, `delete-not-done` and `rename-not-done` details name the resolve key (the glyph, not the `plan:` spelling) and quarry's status.
For a member glyph they add that the package's tree on disk was read with exported, unexported and same-package test-file members and without evaluating build constraints; for `not_found`, the unit's status; for `ambiguous`, each candidate's file, its build constraint and whether the candidates overlap.
A path-shaped target says it was looked up as a file.
Each case names its way forward: add the declaration; check the package path, or the `<pkg>_test` unit for a member declared in an external test package; or declare the member once or give its files disjoint constraints.
**Rationale:** the detail today is `Create target %q still does not resolve`, which hides whether the member or the unit is missing or the answer was ambiguous; #511's `tmuxAllowed` target is #500's member, and the operator could not tell.

### D8 — A partitioned member reads as multipart on the plan side

**Decision:** `planglyph` classifies an `ambiguous` answer as partitioned when every candidate has the same glyph ID, no two candidates sit in one file, and no build configuration selects two candidate files.
The configurations are derived in-process from the candidates themselves: each GOOS and GOARCH named in their filename suffixes or constraints plus one "other" value of each (covering both `unix` and non-`unix`), cgo on and off, and every assignment of the remaining tags they mention; tags a configuration fixes (GOOS, GOARCH, `unix`, `cgo`, compiler and `go1.N` release tags) are never free.
Each file is judged by `go/build.Context.MatchFile` under that configuration.
The classifier fails closed: an unreadable file, an unparsable constraint or more than four free tags leaves the answer `ambiguous`.
A partitioned answer is rewritten to a multipart-shaped answer (status multipart, every candidate as a symbol) in the plan-side callers of `resolveTargets` only, so the Plan-time status policy, the Create inversion, delete ordering and the done-checks treat it as one member declared in several places; `lyx quarry resolve` keeps printing quarry's own answer.
`BindHandles`, which matches the record-batch delta's created IDs rather than a resolve answer, counts a handle bound when any created entry carries its ID, however many carry it.
An overlapping ambiguity stays `glyph-ambiguous` at Plan time and names an overlapping configuration.
**Rationale:** quarry does not evaluate build constraints by design, and every plan-side resolve site already handles multipart correctly: Edit, Delete and Uses pass at Plan time, a Create passes once declared, a Delete blocks until the last body is gone, and an existing member still reads `create-already-exists`.
One classifier at Plan time and record-batch means a glyph passes or fails both for the same reason.

## Constraints

- `PATTERN-completion-signal`: D1 adds no negative verdict over present output files; the tripwire test pinning `wait.go`'s call sites is updated in the same change if those sites move.
- `PATTERN-shuttle-provider-seam`: D1's matcher reads the label text shuttle already receives; label and signal knowledge stays in `claudeengine`.
- `PATTERN-refusal-way-forward`: D5's outcomes and D7's changed details land with their `refusal-spec.md` rows and a test.
- `PATTERN-friction-capture`: D5's `ok: false` envelope is a refusal to the refusal-note writer.
- `PATTERN-no-denied-recovery`: D3 and D6 name no denied command and no slot, strand or prebuilt-binary variable.
- `PATTERN-gate-slot`: D6's compile step runs through `lyx gate test`, never raw `go vet` or `go test`.
- `PATTERN-producer-pointer-rule`, `PATTERN-stencil-ownership`: D6's recovery text points at the plan template's tier paragraph and does not restate it; stencils stay under `contracts/stencils/`, read at call time.
- `PATTERN-spawn-observability`: D8 spawns no process; the configurations come from the candidates, not from `go tool dist list`.
- D8's tests pin the `ambiguous` candidate shape of quarry v0.2.1 (one candidate per declaration, each with its file), so a quarry bump that changes it fails a test.
- Docs in the same commit: `doc.go` of `shuttleengine`, `websterengine`, `webstercli` and `planglyph`; `refusal-spec.md`; `contracts/stencils/claims_test.go` pins for changed stencil sentences.

## Auto-mode assumptions

- D2: Master runs no gate at all; this goes beyond the board's two outcomes for part 1 and is the chair's own pick.
- D4: attribution is run-wide, as for fork writes, not per batch as the board phrases it; a later batch's recovery also reads another batch's recovery leftovers as the run's own, which the suspect-path rule and the card check bound.
- D4: the contract-file check stays on Master and fork writes; no run has shown a recovery writing a contract file.
- D5: `dead` with `recovery_retry: true` exits non-zero too, since that attempt failed even though a retry remains.
- D8: the board's "the plan validator flags such a glyph at Plan time" is read as Plan time and record-batch sharing one classifier; a Create's future file layout is not knowable at Plan time, and D7 names an overlapping one at record-batch.
- D8: the cap of four free tags; real partitioned members use one or two.

## Open risks

- D1: a held gate shell that hangs, or one a fork left whose completion never starts a Master turn, holds Master until `master_timeout_min` (480 in the shipped template), where the verify gate then decides the outcome; the cost is that wait.
- D4: a reset now counts a recovery's tracked writes as the run's own and can discard them, as it already does for fork writes.
- D8: how quarry's delta reports one ID created in two files (two entries, one, or neither) is unverified; `BindHandles` must bind on any of them.
- #511's `rusageStats` cannot be confirmed as a per-GOOS member from base code, where it sits in one file; D7 makes the next occurrence diagnosable either way.

## Acceptance criteria

- A gated Master run whose turn end leaves a `lyx gate test` shell outstanding, with output files present, does not finish at that turn end; the same holds for `cd <dir> && lyx gate test … | tail -5` and for a label with a leading `NAME=value`.
- When the held gate shell completes and Master's next turn ends with nothing outstanding, the run finishes normally through the verify gate.
- An outstanding shell matching no prefix, such as `tail -f`, keeps today's handling: the turn end counts at once.
- A shell outstanding at the run's end appears in the warning, friction note and summary as ended with its result never read.
- A gated run without a re-prompt text on its entry sends today's generic line; Master's verify entry sends the fixer-fork line, and the verify findings file opens with the same instruction.
- A second recovery of a batch lists files the first recovery wrote and left uncommitted under "Written by this run".
- An attaching `recover-batch` call records no new recovery session; a recorded session with no transcript is skipped with a warning and its paths stay under "Not written by this run"; a state without the new field reads as no recovery sessions.
- `recover-batch` answering a terminal `stuck` or `dead` exits non-zero with `ok: false` and the digest fields; on `done` and on a `running` snapshot it exits 0.
- The implementer body names the always-allowed gates; the recovery prompt names the compile-only route for a tagged test.
- A `create-not-done` for a missing member, a missing unit, an overlapping ambiguity and a missing file-path target each names its key, status, candidates where present, and way forward.
- A member declared in `allow_tmux.go` (`//go:build tmux || llm`) and `allow_none.go` (`//go:build !tmux && !llm`) passes a Create done-check and binds its `plan:` handle; one declared in `_linux.go`, `_windows.go` and a `//go:build !linux && !windows` file passes a Plan-time Edit and Delete status check.
- Two candidates selectable together (no constraints, or overlapping ones) stay `ambiguous` with a `glyph-ambiguous` finding naming the overlap; an unparsable constraint stays `ambiguous`.
- A Delete of a partitioned member blocks `delete-not-done` while any body remains.
- `lyx quarry resolve` on a partitioned member still prints `ambiguous`.

## Notes for the plan writer

- D1: `countsAtOnce`, `expiredTurnEnd` and `awaitedLabel` in `internal/shuttleengine/wait.go`; `masterAwaitedShellPrefix` and the Master spawn spec in `internal/websterengine/runlevel.go`; corpus case `claudeengine/testdata/corpus/master-fork-running`; the shell wording in `triage.go` and `summary.go`'s `AppendBackgroundShells`.
  `websterengine` imports `shuttleengine`, so webster's audit command splitter cannot be reused from shuttle.
- D3: `gateRepromptText` in `internal/shuttleengine/gate.go`; `validateSendText` requires one line; `renderVerifyGateFindings` in `verifygatereport.go`.
- D4: `recordedSessions`, `writtenWorktreePaths` in `runwrites.go`; `recoverSpawn` already holds `FindRun`'s `RunState.SessionID`; `reset.go`'s `ownTrackedPaths`.
- D5: `recover-batch`'s cobra `Long` sits near the 2000-byte cap in `cmd/lyx/clitree_test.go`.
- D8: `resolveTargets` also backs `quarrycli/resolve.go` through `planglyph.Resolve`, so the rewrite sits in the plan-side callers; `BindHandles` is in `handle.go`.
  Later resolve consumers on the plan side (`kick-start-pack`, `discussion-premise-cites`) would use the same rewrite.

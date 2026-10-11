## Goal

Close the Webster faults outside the audit that stop or mislead runs (issues #505, #509, #517, #522, #511, #500, #507): background shells cut off at Master's turn end, Master fixing verify failures itself, recovery attribution, exit status and allowed gates, and Create checks that cannot pass for members declared once per build-constraint set.
The record faces the code at base `800a0e892`.

## Scope

In:

1. A spawn-level opt-in in `internal/shuttleengine` that makes every background shell hold a gated run's waiting turn end, set by Master's spawn.
2. A per-entry re-prompt line on shuttle's `GateEntry`, set on webster's `verify` entry.
3. Recovery strands' writes in the run's write evidence (`internal/websterengine`), and `recover-batch`'s exit status for a terminal stuck or dead digest (`internal/webstercli`).
4. One paragraph in the recovery prefix stencil (`contracts/stencils/webster/webster-prefix-recovery.md`).
5. The `create-not-done` and `rename-not-done` new-side finding text (`internal/planglyph`).
6. Build-set ambiguity normalized at planglyph's resolve chokepoint, read by Plan-time validation, record-batch's done-checks and `lyx quarry resolve` alike.

Out:

- Changing quarry (`github.com/Knatte18/quarry`), which by design does not evaluate build constraints.
- Letting a `Create` handle reuse a name whose every declaration the same card deletes (#507's first option); #507 offers its two options as alternatives, and D7 delivers the second.
- A Rename whose old side is a per-build-set member: `renameDeclSource` stays found-only.
- Reviving a dead batch through `lyx loom goto` (#511's third question); at base, `begin-batch`'s report-present refusal already names `lyx webster reset --to batch-start --batch NN`.
- Gate-running mechanics for forks and Master (foreground or background, in-turn waits), which board entry `fork-wait-noise` owns.
- The shell rule of every gated run other than Master's.

## Decisions

### D1. Master's background shells hold its waiting turn end

`Spec.AwaitedShellPrefixes` is replaced by a boolean `Spec` field under which every background shell, of either signal, holds a gated run's waiting turn end exactly as a fork does.
With it set, `countsAtOnce`'s exception never applies: a waiting turn end with every output file present counts as an arrival only through a later turn end with nothing outstanding, or ends by the run's own `Timeout` or the liveness check.
Master's spawn sets it, and `masterAwaitedShellPrefix` goes.
A spec without it keeps today's rule, so every other gated run is unchanged.
Since a done Master outcome can no longer carry an outstanding shell, webster's done-only "Background shells at the run's end" summary section (`AppendBackgroundShells`) is removed; `Result.EndedShells`, the `webster-background-shell` friction note and `RunResult.Warnings` keep recording a shell outstanding when Master's run ends by deadline, liveness or death.

Rationale: a gate shell is only one case of work whose result the run would discard, so a label match on `lyx gate test` would leave other verification shells, and every `cd …` or piped spelling, open to the same fault.
Master's turn-end list also carries shells its forks left running (the `master-fork-running` corpus case records one), so the label says nothing about who started the shell.

Bound: what newly waits is Master's turn end on a shell the old rule let through; it is bounded by Master's `Timeout` and the liveness check, and lyx still kills no shell.
Nothing that held before now passes.

### D2. Webster's verify gate carries its own re-prompt line

`GateEntry` gains an optional re-prompt field, a function of the findings path returning one line; empty keeps `gateRepromptText`'s generic line.
The `verify` entry, assembled where Run builds Master's gate spec, sets a line that keeps the opening `Gate findings recorded at <path> —` and adds only: do not fix it yourself; follow the "A gate failure" section of your instructions with the prompt file `<verify_fix_prompt_path>`.
The procedure stays in the Master stencil, whose "A gate failure" section keeps matching the message by that opening.

Rationale: the re-prompt Master reads at the failure today ends "read it, fix every finding, and end your turn", which contradicts its stencil at the moment it acts.

Bound: the field changes only the text sent; it is validated as a single line like every send, and no gate decision reads it.

### D3. Recovery strands' writes count as the run's own writes

Each recovery spawn appends its provider session id, read from the spawned shuttle run's state, to a list on `BatchState`.
The list carries across a respawn and a re-begin, and survives `reset --to batch-start`, which clears only the count and the start.
`loadRunWrites` audits those sessions and returns their writes in their own `RunWrites` field.
Both readers of the own-path set add that field: `writtenWorktreePaths`, behind the recovery prompt's grouping, and `PlanReset`'s own tracked paths.
The contract-file evidence keeps reading Master's and the forks' writes, not the recovery sessions'.
A recovery session whose transcript cannot be found is skipped with a Warn log, so its writes stay unattributed, instead of failing the audit; a missing Master session keeps failing as today.
A state written before the list existed has no ids, and those recoveries' writes stay unattributed.

Rationale: attribution today is by Master session and its forks, and a recovery strand is a separate session no list records, so a second recovery reads the first one's files as foreign.
The run stays the attribution unit, as it already is for forks.

Bound: attribution widens only to sessions webster spawned and recorded for the run; a path no recorded session wrote stays "Not written by this run".

### D4. `recover-batch` exits non-zero unless the batch ended done

A terminal digest whose status is stuck or dead returns the failure envelope, exit non-zero, carrying the digest fields, `recoveries`, `recovery_retry` and `warnings` the success envelope carries today.
Its message names the way forward: dead with `recovery_retry: true`, `lyx webster recover-batch NN` once more; any other stuck or dead, end the run stuck naming the batch, with the operator's steps `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN`.
A done digest and a running snapshot keep exit 0.
The refusal lands with its row in `contracts/specs/refusal-spec.md`, and, like every `ok: false` webster envelope, leaves a `webster-refusal-recover-batch` friction note, which a failed recovery warrants.
The Master stencil's recover-batch rungs say the stuck and dead digests arrive on that envelope.

Rationale: at base, a failed terminal already exits non-zero through `batch_failed`, and only stuck and dead digests return `output.Ok`, so #522's exit 0 was one of these.
The exit status is what a shell, an operator and Master's completion notification read first.

### D5. The recovery prompt names its always-allowed gates

The recovery prefix gains one paragraph: the `lyx gate test` and `lyx loom lint-comments` steps of the card gates and the batch gate rendered below are always allowed and are not fabric commands;
a card `verify:` step under the `tmux` tag is left to verify, since Publish runs that tier, and the strand does not run it.
The paragraph does not restate how to run a gate, which `webster-body-implementer.md` owns.

### D6. A done-check failure says what the lookup answered

`create-not-done` and the new side of `rename-not-done` name the lookup and its answer: the glyph resolved with quarry against the worktree's current tree, and the status:

- `not_found`, with whether the unit exists, and, when the member resolves in the sibling unit (`<unit>_test` against `<unit>`), that unit's glyph;
- `ambiguous`, with each candidate's file and line, and which D7 condition failed: the build constraints overlap, the candidates span directories or ids, or a candidate's constraint could not be read;
- an unreadable answer, as `glyph-rejected` already reports.

The check, its id and its severity are unchanged.

Rationale: quarry already resolves unexported members and test-file members, so no resolver limit exists to fix; #511's `rusageStats` was a per-GOOS pair, D7's fault, hidden behind "still does not resolve".

### D7. A member declared once per disjoint build set resolves multipart

`resolveTargets`, planglyph's one call to quarry's resolve, rewrites an `ambiguous` answer to `multipart`, with the candidates as its symbols, when all hold:

- every candidate carries the target's own glyph id;
- every candidate sits in one directory, each in a different file;
- no build configuration compiles two of those files, judged as the go tool judges them: the `//go:build` line, or a legacy `// +build` line where no `//go:build` line exists, the file-name `_GOOS`, `_GOARCH` and `_test` suffixes, exactly one GOOS and one GOARCH per configuration, `unix` true exactly for the unix GOOS values, and every other tag free.

The GOOS, GOARCH and unix lists are a static list in lyx, pinned by a test against `go tool dist list`, so the gate spawns no toolchain.
Any other ambiguous answer stays ambiguous, including a candidate file lyx cannot read or whose constraint does not parse.
Every planglyph reader then sees one answer: Plan-time `glyph-ambiguous` passes such a glyph for `Edit`, `Delete` and `Uses`, record-batch's `create-not-done` passes its `Create`, `delete-not-done` blocks while any declaration remains, `create-already-exists` blocks re-creating an existing one, and `lyx quarry resolve` reports every declaration's span.
A glyph that stays ambiguous keeps failing at Plan time as `glyph-ambiguous`, whose detail names the failed condition as D6 does.
A `Create` handle has no declarations at Plan time, so it cannot be flagged then; Plan time and record-batch read one answer by construction, so no glyph Plan accepts fails record-batch on build-set ambiguity.

Rationale: quarry declines to evaluate build constraints because it does not know the target platform; lyx asks whether the declarations can ever meet in one build, which needs no platform.

Bound: the rewrite lets through only same-directory candidate sets the disjointness test proves never compile together; a false "disjoint" would admit a duplicate declaration, which the card and batch gates' compile and the verify gate still reject.

## Constraints

- PATTERN-refusal-way-forward: D4's refusal lands with its refusal-spec row and a test reaching it.
- PATTERN-friction-capture: D4's refusal writes its friction note through the existing `ok: false` path.
- PATTERN-no-denied-recovery: D4's and D5's texts name only commands the agents' settings allow.
- PATTERN-completion-signal: D1 adds no negative finalize, and the AST tripwire stays green.
- PATTERN-shuttle-provider-seam: D1 and D2 stay provider-neutral in `internal/shuttleengine`.
- PATTERN-told-geometry: D3 and D7 read only paths their callers hand them.
- PATTERN-stencil-ownership: D2's line is a Go-built send text like `gateRepromptText` that points at the stencil's procedure, never restates it.
- PATTERN-test-economy and PATTERN-test-speed: cases extend existing table-driven tests where one covers the behavior; new tests stay untagged.
- PATTERN-documentation-lifecycle: the same commit updates `doc.go` of `shuttleengine`, `websterengine`, `webstercli`, `planglyph` and `quarrycli`, `recover-batch`'s and `lyx quarry resolve`'s help text, `contracts/specs/refusal-spec.md`, the Master and recovery stencils, and the stencil claims that pin them.

## Auto-mode assumptions

No operator answered; each pick below took the narrowest reading that closes the reported fault.

- D1 opts Master in rather than changing every gated run, because the board entry and issues report Master only.
- D2 puts the rule in the message, because no verb can spawn an in-session fork.
- D3 keeps the run as the attribution unit, because fork writes are attributed that way already.
- D4 makes a dead digest with `recovery_retry: true` exit non-zero too, because the batch is not done.
- D5 names only the `tmux` tag, because the board entry names only it.
- D7 normalizes in lyx and reaches `lyx quarry resolve`, because quarry's design rules out the change there.

## Open risks

- D1: Master's turn end now waits on a shell that never ends, or on a transcript-signal shell whose completion is never seen, until Master's `Timeout`; the run ends `OutcomeTimeout` instead of `done`.
- D1: a Master `paused` or `stuck` outcome written while a shell runs ends only when the shell does.
- D3: a recovery's transcript lives under the machine-local provider project directory, so on another machine its writes stay unattributed, as fork writes already do.

## Acceptance criteria

- With the D1 field set, a gated run whose waiting turn end has every output file present, started fresh, with only a shell outstanding, keeps waiting; once the shell completes and a turn end has nothing outstanding, the run reaches `Done`; its deadline otherwise ends it `OutcomeTimeout` with the shell in `Result.EndedShells`.
- Without the field, the same turn end still finishes `Done` at once, and an ungated run is unchanged.
- A failed verify-gate evaluation sends Master one line that opens `Gate findings recorded at`, names the fixer-fork prompt path and says not to fix it yourself; another gated run's failed gate still sends the generic line.
- A second recovery of a batch lists the files the first recovery wrote and left uncommitted under "Written by this run", and a file no recorded session wrote under "Not written by this run".
- A recorded recovery session with no transcript is skipped, and the prompt still renders; a state with no recovery session ids renders as before.
- `reset`'s own paths include a recovery strand's tracked writes, and `reset --to batch-start` keeps the session list.
- `recover-batch` exits non-zero with the digest fields, `recoveries` and `recovery_retry` on a terminal stuck and on a terminal dead digest, and writes its friction note; it exits 0 on done and on a running snapshot; the refusal-spec row exists.
- The rendered recovery prompt carries the allowed-gates paragraph.
- A `create-not-done` finding names the status quarry answered, the unit state and any sibling-unit match on `not_found`, and each candidate's file and the failed condition on `ambiguous`.
- Build-set normalization, in planglyph's resolve:
  a `//go:build tmux || llm` and `//go:build !tmux && !llm` pair resolves multipart;
  a `_linux.go` and `_windows.go` pair resolves multipart;
  a `_linux.go` and `//go:build !linux` pair resolves multipart;
  a legacy `// +build linux` and `// +build !linux` pair resolves multipart;
  a `_linux.go` and `//go:build unix` pair stays ambiguous;
  two `//go:build ignore` files stay ambiguous;
  a test file beside a non-test file stays ambiguous;
  two declarations in one file stay ambiguous;
  candidates in two directories that collide on one unit stay ambiguous;
  a malformed constraint or an unreadable candidate file stays ambiguous.
- A plan card that edits a per-GOOS member by glyph passes `lyx loom validate-plan`, a batch that creates a member once per disjoint tag set passes record-batch's done-checks, and a Delete leaving one of the declarations fails `delete-not-done`.

## Notes for the plan writer

- `go/build/constraint` parses both constraint forms; `go/build`'s `Context.MatchFile` evaluates one file for one configuration, file-name suffixes included.
- The recovery's session id is on the shuttle run state `recoverSpawn` already reads through `shuttleengine.FindRun`.
- Parallel editors with no ordering dependency: `fork-wait-noise` (the Master template and `webster-body-implementer.md`, which recovery strands share), `shuttle-startup-ready` and `parent-review-wait` (`internal/shuttleengine/wait.go`), and `plan-glyph-tuning` (`internal/planglyph` and record-batch).
- `proc-child-lifetime` edits per-GOOS members in `internal/proc` and `internal/gatecli` by glyph, which needs D7.

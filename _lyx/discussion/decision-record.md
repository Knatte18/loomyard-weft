# Decision record — dbench-ohom: Webster stability (gate shells, Master self-fix, recovery, Create checks)

## Goal

Close the Webster faults outside the audit that stop or mislead runs:
a `lyx gate test` shell cut off at Master's turn end,
Master fixing a verify failure itself,
a second recovery treating the first recovery's files as foreign and `recover-batch` exiting 0 over a failed batch,
a recovery prompt that leaves its allowed gates unclear,
and Create, Edit and Delete targets that can never pass for a member declared once per mutually exclusive build-constraint set.

## Scope

In:

1. Master's turn end waits on every outstanding `lyx gate test` shell (#505, #509, #517).
2. The verify gate's re-prompt to Master sends it to the fixer fork, never to fixing (board item 2).
3. Recovery strands' writes count as the run's own, and `recover-batch` exits non-zero on every non-done terminal digest (#522).
4. The shared implementer body names the gates a fork or recovery may always run (board item 4).
5. The `create-not-done` and `rename-not-done` (new side) details say how the target was looked up (#511).
6. planglyph treats an ambiguous answer whose candidates sit in mutually exclusive Go build contexts as multipart, at Plan time and at record-batch (#500, #507).

Out:

- A Create that reuses a name the same card Deletes (#507's second half): replaced by Decision 6's Edit-by-glyph route.
- Teaching quarry (external module `github.com/Knatte18/quarry`) build constraints, and any change to `lyx quarry resolve`'s raw answer.
- Reviving a dead batch through `lyx loom goto` (#511 point 3): not in the board entry.
- A shuttle-wide awaited-shell default for other gated rows.
- Renaming a per-GOOS symbol through a re-sign handle: `renameDeclSource` stays found-only.
- Killing a provider shell: lyx has no pid for it (see board entry `proc-child-lifetime`).

## Decisions

### D1 — Master awaits every `lyx gate test` shell

Decision: Master's shuttle spec adds the prefix `lyx gate test` to `AwaitedShellPrefixes` beside `lyx webster recover-batch`.
A Master turn end with such a shell outstanding, Master's own or a fork's shell surfacing on Master's session, never counts at once, whether or not both output files exist.
Bound: what can now keep Master's run waiting is any shell whose command text starts with `lyx gate test`.
`lyx gate test` has no timeout of its own below `master_timeout_min` (only its slot wait and go test's per-binary timeout), so `master_timeout_min` and the liveness check bound the hold, unlike `recover-batch`, which `recovery_timeout_min` bounds.
A gate command that does not start with the prefix (for example `cd x && lyx gate test`) is not awaited and is only recorded, as today.
The `runlevel.go` comment on the prefix and `websterengine/doc.go`'s "Background shells in Master's wait" section say both bounds.
`Result.EndedShells`, the run warning, the friction note and the summary section stay as they are for a deadline or liveness end, and their wording says the shell's result was not read.
Rationale: the label shuttle matches is the Bash command text, so the prefix covers the shells in the issues without new engine logic.
The run outcome is never demoted: a done run still passed the verify gate's full plan verify, so a discarded gate shell's result is redundant, not missing.

### D2 — The verify gate's re-prompt carries its own text

Decision: `shuttleengine.GateEntry` gains an optional caller-supplied re-prompt text function of the findings path.
A nil function keeps today's `gateRepromptText` wording for every other gated row.
Webster's verify gate entry supplies a one-line text that opens with the unchanged `Gate findings recorded at <path>` and continues: do not edit, commit or test anything yourself; read the file once, spawn one fixer fork as "A gate failure: spawn one fixer fork" says, and end your turn.
The verify-gate report (the findings file Master reads once) gains a first line saying its findings are for the fixer fork, not for Master.
Bound: the field changes wording only.
The one-line send validation, the entry's attempts and every gate verdict are unchanged, and the text is caller data on `GateEntry`, provider-neutral (`PATTERN-shuttle-provider-seam`).
Rationale: the root cause is Go's generic re-prompt telling Master "fix every finding", against its stencil.
The rule now sits in both places Master reads a failure.

### D3 — Recovery sessions count as the run's own writes

Decision: when recover-batch spawns a recovery strand, it resolves the strand's session id (through `shuttleengine.FindRun` on the strand, as Master's spawn does) and appends it to a new `BatchState` list of recovery session ids.
The list carries across a respawn and a re-begin, the way `Recoveries` does, and is persisted with the spawn's state save.
`recordedSessions`/`loadRunWrites` audit those sessions too and file their writes in a third `RunWrites` slice for recovery writes, never in `RunWrites.Master`.
The recovery prompt's "Written by this run" grouping and reset's own-path set read the recovery slice beside Master's and the forks'.
Contract-file evidence (`contractevidence.go`) keeps reading `RunWrites.Master` alone.
Bound: attribution widens only to sessions recover-batch itself spawned and recorded in this run's `state.json`; `run --fresh` archives them with the state.
A session id that cannot be resolved at spawn is recorded as absent and that recovery's writes stay unattributed, as today.
Rationale: today no recovery session is ever audited and each recovery replaces the batch record, so a second recovery sees the first one's files as foreign.
The id is read at spawn, because a done recovery removes its run directory.

### D4 — `recover-batch` exits non-zero on a non-done terminal digest

Decision: `recover-batch` exits 0 only for a done digest or the `--wait` running snapshot.
A stuck, dead or failed terminal digest exits non-zero through `output.ErrFields`, its envelope carrying the same digest fields (`status`, `recoveries`, `recovery_retry` and the rest) and an error message naming the way forward: the failure-ladder rung for the status, and `lyx webster recover-batch NN` when `recovery_retry` is true.
It is a webster refusal under `PATTERN-refusal-way-forward`: its row lands in `contracts/specs/refusal-spec.md` under `## webster` and a test reaches it, in the same commit.
The Master stencil gains one sentence: a non-done recover-batch digest exits non-zero and is still read by its fields.
Its rungs keep keying on `status` and `recovery_retry`.
Rationale: the board entry asks for a non-zero exit when the batch ends failed and terminal; one rule for every non-done digest keeps the exit code independent of `recovery_retry`.

### D5 — The implementer body names the always-allowed gates

Decision: the shared implementer body (`webster-body-implementer.md`), beside its fabric ban, states once: the card gate, the batch gate, a card's `verify:` lines and `lyx loom lint-comments` are always allowed and are neither fabric nor webster verbs.
It states that a `tmux`-tier test is never run by a card or batch, by plan policy, since Publish runs that tier, and names no raw `go` command for it (`PATTERN-no-denied-recovery`).
Rationale: the recovery prompt is the recovery prefix plus this body, so one statement serves fork and recovery alike.

### D6 — Build-exclusive ambiguity reads as multipart in planglyph

Decision: one chokepoint in `internal/planglyph` rewrites a quarry `ambiguous` answer to `multipart` (its candidates moved to symbols) when every candidate is a `.go` file and no real build context matches two candidate files.
Every quarry status consumer in planglyph reads the rewritten answer: the plan gate's `statusFindings`, `createFindings`, `DoneChecks`, the handle checks, `deleteorder`, drift and scope.
The check uses `go/build`'s `Context.MatchFile` with an injected `OpenFile` reading the candidate's `file` joined to the told worktree root, so the go command's own filename-suffix, `unix` and `//go:build` rules apply and planglyph reimplements none of them.
It tries a superset of the real build contexts:
GOOS and GOARCH values drawn from every atom in the candidates' constraints and filename suffixes, plus one GOOS value no atom names on each side of `unix` and one GOARCH value no atom names;
every other tag, `cgo` and release tags (`go1.N`) included, as a free tag in every subset.
Two files are disjoint when no tried context matches both.
Bound, fail-closed (the answer stays ambiguous): an unreadable candidate file, a constraint that does not parse, a non-Go candidate, more than 12 free tags across a pair, or a candidate file no tried context matches.
A same-name pair across `package x` and `package x_test`, which quarry merges under one glyph, has no build-constraint disjointness and stays ambiguous.
Reading the file quarry's answer names converts no glyph, so `PATTERN-glyph-conversion-chokepoint` holds.
`lyx quarry resolve` keeps quarry's raw answer: quarrycli's contract is that it never re-shapes an answer.
Effects:
a per-tag Create passes `create-not-done`;
an Edit, Uses or Delete target of a per-GOOS or per-tag symbol validates by glyph at Plan time;
a Create over a name already declared per build tag reports through the multipart arm of `create-already-exists`, whose wording must fit that case;
`delete-not-done` still requires `not_found`, every declaration gone;
duplicates with overlapping constraints stay `glyph-ambiguous` at Plan time.
Rationale: quarry is an external module, and planglyph already reads multipart as one symbol the language declares in several places, which a per-build-context member is.
Normalizing at one site keeps the plan gate and record-batch agreeing, so an ambiguity is caught at Plan time rather than at record-batch.

### D7 — The Create and Rename done-check details say how the target was looked up

Decision: the `create-not-done` detail and `rename-not-done`'s new-side detail name the resolve key looked up and quarry's status (after D6's rewrite).
For `not_found` they say whether the unit or only the member is missing (from the answer's `Unit`) and that the lookup covers unexported and `_test.go` declarations, as a claim about the lookup.
For `ambiguous` they list each candidate's file and say the candidates' build constraints overlap or could not be read.
`delete-not-done`'s detail already prints the status and is unchanged.
The refusal-spec row for `create-not-done` is updated in the same commit.
Rationale: #511's targets were not unexported-identifier failures; the tree shows `cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed` resolve `found` today, and the old detail printed "still does not resolve" for an ambiguous answer.

## Constraints

- `PATTERN-shuttle-provider-seam`: D1's prefix and D2's re-prompt are caller data on `Spec`/`GateEntry`; nothing provider-specific enters `shuttleengine`.
- `PATTERN-completion-signal`: D1 adds no negative-verdict site in `shuttleengine`.
- `PATTERN-refusal-way-forward`: D4's non-zero exit and D7's changed details name a way forward, carry a `refusal-spec.md` row and are reached by a test, in one commit.
- `PATTERN-no-denied-recovery`: D5 names no command the agents' settings deny.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`: stencil edits are text edits read through `stencilstore`; no stencil asks an agent to load a skill or ask the operator.
- `PATTERN-glyph-conversion-chokepoint`: D6 reads files by the path quarry's answer names and derives no path from a glyph.
- `PATTERN-told-geometry`: D6's worktree root is the one planglyph is already told.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`: new scenarios extend existing table tests where one covers the surface; a git-spawning test runs under `gitkit.HermeticGitEnv()`.
- Test budget: a package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.
- Docs in the same commit: `internal/shuttleengine/doc.go` (D2), `internal/websterengine/doc.go` (D1, D3, D4), `internal/webstercli` help and doc for `recover-batch` (D4), `internal/planglyph/doc.go` (D6, D7).
- Markdown and Go comments use semantic line breaks.

## Auto-mode assumptions

- The shells in #505, #509 and #517 may be forks' shells surfacing on Master's session rather than Master's own; D1 covers both, so the provenance was not confirmed from a run's `EndedShells`.
- Why the quick-recipe recovery could only compile-check tmux-tagged steps is not visible in the code; D5 states the plan-policy rule rather than a cause.
- The `#522` exit-0 case is taken to be a stuck or dead terminal digest going through `output.Ok`; D4's rule covers every non-done digest whatever the exact path was.
- 12 free tags per pair is enough for every real constraint pair in the tree.

## Open risks

- D1 can hold Master for up to `master_timeout_min` behind a hung `lyx gate test` shell, with no shorter bound.
- D6's superset of contexts can still read a real disjoint pair as overlapping (fail-closed, stays ambiguous); it must never read an overlapping pair as disjoint.
- D6 moves candidates into symbols for every planglyph consumer, so delete-order findings for a per-GOOS Delete change; consumers that assume one file per multipart symbol need checking.
- D4 changes the exit code Master's background notification reports; a Master that reads the exit code instead of the fields could misroute until the stencil sentence lands.

## Acceptance criteria

- A gated Master run's waiting turn end, with both output files present and an outstanding shell whose label starts with `lyx gate test`, does not count; with only unawaited shells outstanding on a fresh first arrival it still counts.
- A run that ends on its deadline with a `lyx gate test` shell outstanding records it in `EndedShells` and the warning says its result was not read.
- A gated run whose verify entry supplies a re-prompt text sends that text; a gate entry with no re-prompt text sends today's wording unchanged.
- Webster's verify-gate re-prompt opens with `Gate findings recorded at` and tells Master to spawn the fixer fork, not to fix; the verify-gate report's first line addresses the fixer fork.
- A recovery strand's successful writes appear under "Written by this run" in a later recovery's prompt, including after the first recovery was replaced by a second spawn.
- A recovery strand's write to a Master contract path does not count as Master contract evidence.
- `recover-batch` exits non-zero for a stuck, dead and failed terminal digest, its envelope carrying `status`, `recoveries` and `recovery_retry`; it exits 0 for a done digest and the running snapshot.
- The rendered implementer body (fork and recovery) names the card gate, batch gate, card `verify:` lines and `lyx loom lint-comments` as always allowed and the tmux tier as left to Publish.
- planglyph reads as multipart: a GOOS filename-suffix pair; `//go:build tmux || llm` against `//go:build !tmux && !llm`; `linux` against `!linux`.
- planglyph keeps ambiguous: `linux` against `unix`; an unconstrained file against `tmux || llm`; an unparsable constraint; a non-Go candidate; an `x`/`x_test` same-name pair; an unreadable candidate file.
- `DoneChecks` passes a Create whose target is declared once per exclusive build context and still fails a Delete while any declaration remains; the plan gate passes an Edit target declared per GOOS.
- The `create-not-done` detail names the key, the status, unit-versus-member for `not_found` and each candidate file for `ambiguous`; `rename-not-done`'s new side does the same.
- `lyx quarry resolve` output is unchanged for an ambiguous per-GOOS symbol.

## Notes for the plan writer

- Item 1's wait logic is `shuttleengine/wait.go` `expiredTurnEnd`/`countsAtOnce`; Master's spec is in `websterengine/runlevel.go`.
- Item 2's generic text is `shuttleengine/gate.go` `gateRepromptText`; the Master stencil and `contracts/stencils/claims_test.go` key on its opening.
- Item 3's attribution is `websterengine/runwrites.go`; `BatchState` notes `SessionID` is empty for a recovery record.
- `TestDoneCheckVerdicts_Rules` in `planglyph/donecheck_test.go` is the table to extend for D6/D7's done-check rows.
- `lyx webster validate` runs planglyph's checks against the tree; check it reaches D6's rewritten answer.

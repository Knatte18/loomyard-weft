# Decision record — Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close the six Webster faults outside the audit that stop or mislead runs:
a run that finishes while a background gate shell is still running, Master fixing a verify failure itself, a second recovery disowning the first recovery's files, a recovery prompt that leaves its allowed gates unclear, a Create check whose refusal cannot be told from a resolver limit, and build-tagged or per-GOOS members that resolve `ambiguous` and so can never pass a Create, Edit or Delete.

## Scope

In:

- shuttle's turn-end rule for a background gate shell, and the record of every shell outstanding when a run ends;
- the gate re-prompt text Master reads when the plan-level verify fails;
- recovery attribution of an earlier recovery's uncommitted paths, and `recover-batch`'s exit code on a terminal non-done digest;
- the allowed-commands statement in the shared implementer body that the fork and recovery prompts compose;
- the detail text of `create-not-done`, `delete-not-done`, `rename-not-done`, `glyph-not-found`, `glyph-ambiguous` and `create-already-exists`;
- a build-constraint-aware reading of an `ambiguous` resolve answer in `internal/planglyph`, applied by every resolve policy there;
- the `doc.go`, refusal-spec rows, stencil claims tests and test-budget rows those changes touch.

Out:

- changing quarry: it is a separate module (`github.com/Knatte18/quarry`), and its resolve contract is syntactic;
- letting `lyx loom goto` into Webster revive a dead batch (issue #511's third ask): not in the board entry;
- letting a Create handle reuse a name the same card deletes (issue #507's first option): unnecessary once a per-GOOS symbol is editable and deletable by glyph;
- hunting and killing a background shell's process by PID: shuttle has no PID for a provider-run shell, and the strand's removal already ends it;
- any change to the run-exit audit, to `accept-audit`, or to the audit gaps the previous commit closed.

## Decisions

### D1. A `lyx gate` background shell holds the turn end, shuttle-wide

A background shell whose label opens with `lyx gate`, or carries `lyx gate` right after a shell separator (`&&`, `;`, `|`), is awaited by every shuttle run exactly as a fork or an `AwaitedShellPrefixes` match is: the turn end waits on it whatever the output files say and whichever signal reported it, bounded only by the run's `Timeout` and the liveness check.
The set of built-in awaited prefixes is one constant in `internal/shuttleengine`, beside the spec's caller-supplied list, and the two are unioned at the one site that decides whether a turn end holds.

Rationale: a gate run is verification the agent has not read yet, and a `done` reached across it is the fault in #505, #509 and #517.
`lyx gate` is lyx's own CLI, so the rule is lyx knowledge, not provider knowledge, and belongs in shuttle rather than in each caller; `BackgroundTask.Label` is already provider-neutral, so the provider seam is untouched.

Bound: the hold matches on the label only, so a gate run started through a wrapper that does not name `lyx gate` is not held; the hold never outlasts the run's `Timeout`.

### D2. Every shell outstanding at a run's end is recorded as discarded, and the webster run says so

`Result.EndedShells` gains a `Discarded` mark, true on every shell the run's end cut off rather than waited out, with the outcome that ended the run.
The strand's removal remains the stop; no PID is hunted.
Webster's run-level handling keeps writing its friction note from `EndedShells` and additionally names each discarded gate shell in the run's recorded background-shell record, so the record never reads as a gate that passed.

Rationale: with D1, a gate shell can no longer be cut off by a `done`; what remains is a `timeout`, `dead` or stop that ends it, and the operator must be able to see that the gate's result was never read.

Bound: a record and a note only; it changes no outcome classification.

### D3. The gate re-prompt text is caller-supplied per entry; webster's names the fixer fork

`GateEntry` gains an optional re-prompt instruction.
Absent, shuttle's message stays as today: read the findings file, fix every finding, end the turn.
Webster's verify entry supplies: do not fix it yourself; spawn one fixer fork on the rendered verify-fix prompt file, as the stencil's "A gate failure" section says, then end the turn.
The message names the stencil section and the prompt file path; it does not restate the section.

Rationale: the Go-rendered message is where Master reads a verify failure, and today it literally says "fix every finding", which is what Master did in `driver-context`.
Go cannot spawn an in-session fork, so a webster verb that spawns the fixer is not available; the text is the lever.

Bound: text only; attempts, gating and verdicts are unchanged.

### D4. Recovery attribution is by batch: every recovery session of the batch is the run's own

A batch record carries the session id of every recovery strand webster spawned for it, carried across a respawn and a re-begin the way `Recoveries` is, and the run's write history audits those sessions beside Master's.
A second recovery's prompt therefore lists an earlier recovery's uncommitted paths under "Written by this run", keeps the suspect-path rule for anything the failure digest names, and the "Not written by this run" group holds only paths no session of the run wrote.

Rationale: the first recovery's edits are card work the batch owns; disowning them blocked `quick-recipe` batch 01 and needed an operator ruling.

Bound: only sessions webster spawned for that batch count; a path no run session wrote stays foreign.

### D5. `recover-batch` exits non-zero on any terminal digest that is not `done`

A terminal `stuck`, `dead` or `failed` digest keeps every digest field in the envelope, including `recovery_retry` and `recoveries`, and exits non-zero with an error message naming the ladder rung that applies.
The refusal spec gains the row, and the Master stencil's failure ladder gains a `status: failed` rung, read like `stuck`, and says the rungs are read from the fields whatever the exit code.

Rationale: an exit 0 with a failed batch misleads a caller that reads the code first, and record-batch already exits non-zero when a batch is rejected on its merits.

Bound: the exit code and message; the digest and state persisted are unchanged.

### D6. The implementer body states what a strand may always run, and what it leaves to verify

The shared implementer body's command section says, in one place the fork and recovery prompts both compose:
`lyx gate test` on the batch's packages, standard and `integration` tags, `lyx loom lint-comments`, every card `verify:` line and the batch gate are always allowed, and the sandbox rule bans only mutating `lyx fabric` verbs and clones of the hub's remotes.
A card step or verify line tagged `tmux` or `llm` is not run by the strand: the strand compile-checks it and names it in the report as left to the plan-level verify gate.
The recovery prefix adds nothing of its own on this; it composes the body.

Rationale: the recovery in `quick-recipe` could not tell whether the gates fell under the fabric ban; stating it where the ban lives keeps one statement.

Bound: the allowance names lyx's own gate verbs and the card's own lines; it opens no raw module-wide or tiered `go` run.

### D7. Done-check and resolve refusals name the lookup; no resolver change for unexported or test-file members

Exploration confirmed quarry resolves unexported members and members declared in `_test.go` files, with an external test package under the unit `<dir>_test`.
No resolver limit exists there, so no code changes for it.
Every finding detail from a resolve answer names what was looked up and how: the resolve key, the unit it was looked up in, the status answered, the candidate files on `ambiguous`, and on `not_found` whether the unit itself resolved, with a sentence naming the `<dir>_test` unit rule when the unit resolved and the member did not.

Rationale: both refusals in #511 were build-constraint cases, not an unexported limit, and nobody could tell from the text.

Bound: text only.

### D8. An `ambiguous` answer whose candidates sit under pairwise exclusive build constraints reads as `multipart`

`internal/planglyph` gains one classifier over an `ambiguous` answer's candidate files: it reads each file's `//go:build` expression and its GOOS and GOARCH filename suffixes the way the toolchain does, and evaluates the expressions with `go/build/constraint` over every assignment of the tags they mention, GOOS names and GOARCH names each one-hot.
When no assignment satisfies two files at once, the answer is "split by build constraint" and every policy in the package reads it exactly as `multipart`: an Edit or Delete target passes at Plan time, a Create target is `create-already-exists`, a Create or Rename-new done-check is resolved, and a Delete or Rename-old done-check is still-present.
A set in which any file carries no constraint, in which two constraints overlap, or whose tag universe exceeds a small fixed bound stays `ambiguous` and blocks at Plan time as today, so a truly ambiguous Edit or Delete glyph is flagged before any batch runs.

Rationale: `tmuxAllowed` in #500, `StartTime` and `runChild` in #507 and both targets in #511 are one symbol declared once per exclusive constraint set, the idiomatic Go shape, and quarry's syntactic answer is the correct raw input for lyx's policy layer to interpret.

Bound: only a candidate set fully covered by disjoint constraints is admitted; nothing else about the four-value vocabulary changes, and `Status.Known()` still fails closed.

### D9. Docs land in the same commit

`internal/shuttleengine/doc.go` records D1 and D2, `internal/websterengine/doc.go` records D3, D4 and D5, `internal/planglyph/doc.go` records D7 and D8, the refusal spec gains D5's row and D7's detail wording, and the stencil files carry D3's pointer and D6's section.
No new PATTERN entry: each rule is a module rule stated in its module's `doc.go`.

## Constraints

- `PATTERN-shuttle-provider-seam`: D1 reads `BackgroundTask.Label`, which is provider-neutral; nothing provider-specific enters `shuttleengine`.
- `PATTERN-completion-signal`: D1 changes when a turn end counts, not how a negative completion answer is finalized; `allOutputFilesExist` is still consulted first on that path.
- `PATTERN-friction-capture`: D2 keeps the Go-authored friction note for ended shells.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D5's non-zero exit and D7's reworded details each land with their spec row and a test that reaches them; every way forward named is a verb the agents may run (`lyx webster recover-batch NN`) or an operator step.
- `PATTERN-producer-pointer-rule`: D3's message and D6's section point at the stencil contract; neither restates it. D6 lives in the body once; the recovery prefix does not repeat it.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`: the stencil edits keep every rendered directive and ask no agent to load a skill or ask the operator.
- `PATTERN-stencil-ownership`: stencils stay read from the told directory; no embedded bytes.
- `PATTERN-sole-parsers`, `PATTERN-ref-shape-registry`: D8 reads quarry answers inside `planglyph`; it parses no plan file and classifies no ref shape.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-testkit`: each scenario below extends an existing table where one covers the surface; D1 and D2 pin at shuttle's public `Wait` surface with injected clocks; D8's constraint cases are unit-tier with fixture files, no quarry repo needed where a `ResolveResult` can be handed in.
- Test budget: a package whose new top-level tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists the file under `Edit:`.
- `PATTERN-markdown-link-integrity`, `PATTERN-comment-line-breaks`: docs and comments follow them.

## Auto-mode assumptions

- A1. The gate shells in the three issues were labelled with their command text, so a label-prefix match reaches them; Plan confirms what the Stop payload and the transcript put in `Label`.
- A2. Webster can learn a recovery strand's session id from the shuttle run it spawned, at or after spawn, so D4 has a value to carry.
- A3. A `tmux`-tagged step cannot run inside a strand's pane because the tier's harness conflicts with reed's own tmux; the board's expectation is followed and the reason is confirmed at Plan time.
- A4. `go/build/constraint` plus the toolchain's filename-suffix rule cover every build-constraint form in this repository; `// +build` lines are not supported and such a file reads as unconstrained.
- A5. The `status: failed` terminal digest is reachable from `recover-batch`'s success path today; if Plan finds it unreachable, D5 covers `stuck` and `dead` and the rung is still added for symmetry with `record-batch`.
- A6. A `done` outcome reached with no gate shell outstanding and other shells outstanding keeps today's rule; nothing in the issues asks to hold on an arbitrary shell.

## Open risks

- R1. A transcript-only gate shell whose completion the transcript never shows holds the turn end until the run's `Timeout`; Plan checks how completion is observed for each signal and, if the transcript cannot show it, D1 falls back to the payload signal with the transcript case recorded under D2.
- R2. Holding Master's turn end on a gate shell lengthens runs where Master starts a gate it never meant to read; the bound is the run `Timeout`, which already applies to forks.
- R3. D4 widens the "Written by this run" set; a recovery session that wrote outside its card is now the batch's own, which the suspect-path rule and the post-batch pass still check.
- R4. D8 admits a candidate set as one symbol on constraint evaluation alone; a wrong one-hot list for GOOS or GOARCH would admit an overlap.
  The lists come from the toolchain's own tables, not hand-typed.
- R5. D5 changes an exit code the Master stencil's ladder reads; the stencil edit and the claims test land in the same commit.

## Acceptance criteria

- AC1. A waiting turn end with a `lyx gate test` shell outstanding, every output file present, holds; the run finishes only after the shell's completion is observed, or at `Timeout`.
- AC2. The same with a `lyx gate test` shell after `cd x &&` holds; one with an unrelated shell outstanding and every output file present finishes as today.
- AC3. A run ended by `Timeout` with a gate shell outstanding reports it in `EndedShells` marked discarded, and webster's record and friction note name it.
- AC4. A gate entry with no re-prompt text sends today's message; webster's verify entry sends a message that names the stencil section and the verify-fix prompt path and says not to fix it in place.
- AC5. A second recovery of a batch whose first recovery left uncommitted paths renders them under "Written by this run"; a path no run session wrote still renders under "Not written by this run".
- AC6. `recover-batch` on a terminal `stuck`, `dead` or `failed` digest exits non-zero with the digest fields and an error naming the ladder rung; a terminal `done` exits 0.
- AC7. The rendered fork and recovery prompts both carry the allowed-commands statement once, and the stencil claims test pins it.
- AC8. `create-not-done` on an `ambiguous` answer with overlapping constraints names the key, unit, status and candidate files; on `not_found` with a found unit it names the `<dir>_test` unit rule.
- AC9. An `ambiguous` answer over `//go:build tmux || llm` and `//go:build !tmux && !llm` files, and one over `_linux.go` and `_windows.go` files, passes an Edit and a Delete at Plan time, is `create-already-exists` for a Create, resolves a Create done-check and keeps a Delete done-check blocking.
- AC10. An `ambiguous` answer with one unconstrained file, or with `//go:build linux` and `//go:build !windows`, stays `glyph-ambiguous` at Plan time and `create-not-done` at record-batch.
- AC11. The refusal spec has a row for D5, the three `doc.go` files describe the new rules, and `lyx gate test ./...` plus the integration tier are green.

## Notes for the plan writer

- Shuttle's turn-end decision, `AwaitedShellPrefixes`, `EndedShells` and the gate re-prompt text all live in `internal/shuttleengine`; webster's Master spawn supplies its awaited prefix and gate entry in `internal/websterengine`'s run-level code, which also writes the background-shell friction note.
- The run's write history is read in `internal/websterengine`'s run-writes reader from the sessions the state records; the recovery prompt's path grouping sits beside the recovery spawn.
- The resolve policies are `resolve.go`, `create.go` and `donecheck.go` in `internal/planglyph`; quarry's `Symbol.File` carries each candidate's repository-relative path, which is what D8's classifier reads.
- The implementer body and the master template are under `contracts/stencils/webster/`, with a claims test over the stencil set.

## Goal

Close six Webster faults that stop or mislead runs outside the audit, as the board entry `twb-tw-om-2` lists them:
a background gate shell cut off by a run that ends `done` (#505, #509, #517),
Master fixing a verify failure itself,
a second recovery treating the first recovery's files as foreign and `recover-batch` exiting 0 on a batch that did not finish (#522),
a recovery prompt that does not name its allowed gates,
a `create-not-done` refusal that does not say how it looked the target up (#511),
and members declared once per build-constraint set or per GOOS that resolve `ambiguous` and so can never pass a Create, Edit or Delete check (#500, #507).

## Scope

In:

- `internal/shell`: the quote-aware command-segment splitter, moved from `internal/fabricengine/refscanner.go` so the audit classifier and shuttle share one.
- `internal/shuttleengine`: awaited-prefix matching at segment starts, and an optional per-entry re-prompt text on `GateEntry`.
- `internal/websterengine`: Master's awaited shell prefixes, the verify-gate entry's re-prompt text and dirty-tree findings wording, and recovery sessions in the recovery prompt's write evidence.
- `internal/webstercli`: `recover-batch`'s exit code and its help text.
- `contracts/stencils/webster/`: the recovery prefix's gate paragraph, and two Master template lines (a background `lyx gate` result; `recover-batch`'s exit code).
- `internal/planglyph`: one Go build-constraint disjointness rule used by every positive reading of an `ambiguous` answer, and the `create-not-done` detail.
- Docs in the same commit: shuttleengine, websterengine and planglyph `doc.go`, `recover-batch`'s help, `contracts/specs/loom-plan-spec.md` where it states the ambiguity rule, and `PATTERN.md` plus its background for D1's widened seam entry.

Out:

- Changing quarry itself (`github.com/Knatte18/quarry`, an external module).
- A `Create` handle that reuses a name every declaration of which the same card deletes (#507's first option).
- Reviving a dead batch through `lyx loom goto` (#511's third point), which the board entry does not ask for.
- How agents run gates (in-turn waits, foreground quick checks): `fork-wait-noise` owns that; this task owns only what a running gate shell does to Master's turn end.
- Moving `lyx gate`'s child lifetime into `internal/proc` (`proc-child-lifetime`).

## Decisions

### D1. Master's gate shells hold its turn end

Master's spawn declares `lyx gate ` as a second awaited shell prefix beside `masterAwaitedShellPrefix`, so a background `lyx gate` shell holds Master's turn end exactly like a fork, on every arrival, the first fresh one included.
Shuttle's `awaitedLabel` matches a prefix at the start of any command segment of the label, where segments come from the shared quote-aware splitter in `internal/shell`; its boundaries cover newline, `&&`, `||`, `;`, `|`, `&`, `(` and `$(`, and a separator inside quotes is not a boundary.
`PATTERN-shell-mechanics-seam` widens from building pane-shell command strings to building and splitting them, in the same commit, and `internal/fabricengine`'s audit classifier calls the moved splitter instead of its own.
Rationale: `expiredTurnEnd` finishes the first fresh gated arrival at once when only unawaited shells are outstanding, the path that discarded the integration tiers in #505, #509 and #517; holding lets the shell's completion notification start Master's next turn.
The hold applies whatever started the shell, since Master's stencil says it never runs verify yet all three issues show it did.
Bound: the hold lets nothing through; it can only keep Master's run waiting longer, bounded by the run's deadline, the liveness check and `go test`'s own per-binary timeout.
Every shell no prefix matches keeps today's handling: `Result.EndedShells`, the `webster-background-shell` friction note and the `RunResult.Warnings` entry.
Master's template gains one rule: a background `lyx gate` result is information only; Master never edits or commits on it, and the verify gate decides.

### D2. The verify-gate re-prompt tells Master to fork the fixer

`GateEntry` gains an optional re-prompt text builder taking the findings path; the failing entry's builder chooses the line, and an entry without one keeps `gateRepromptText`'s generic line, so other gated callers are unchanged.
Webster's verify-gate entry sets a one-line builder, a Go string in `internal/websterengine` beside the entry, as `gateRepromptText` is in shuttle: it names the findings file, says Master does not edit or commit, and says to spawn the one verify-fix fork with the prompt file at `verify_fix_prompt_path` as the template's "A gate failure" section says, then end the turn.
The dirty-tree findings text (`renderVerifyGateFindings`) addresses the fixer fork, not Master, when it asks for paths to be committed or removed.
Rationale: today's line, `Gate findings recorded at … — read it, fix every finding, and end your turn.`, tells Master to fix the findings itself at the moment it reads a failure, which is what `driver-context` followed.
The text is a one-line message, not a prompt file, so `PATTERN-stencil-ownership` does not apply; it points at the template section and the fixer's prompt file instead of paraphrasing them (`PATTERN-producer-pointer-rule`).
Bound: the builder chooses text only; the line still passes `validateSendText`'s one-line check, and the entry's attempt budget, failure count and verdict are unchanged.
Webster's attach path builds the same entry, so an attached Master receives the same line.

### D3. A recovery's own writes count as the run's in the recovery prompt

Each recovery spawn records its provider session id on the batch record, in a list carried forward across a re-begin, a respawn and a Webster restart that reclaims the strand, the way `Recoveries` is.
The recovery prompt's write evidence (`UncommittedPaths`) audits those sessions after the Master sessions and collects each one's own write events, so an earlier recovery's files read "Written by this run".
A recorded recovery session whose transcript is missing contributes no writes and is logged at Warn; it is not an error, so the next recovery and every reset still run.
Reset's own-path set keeps reading Master and fork sessions only.
Rationale: attribution covers Master sessions and their forks only, so the first recovery's 19 uncommitted files in `quick-recipe` read as foreign and the second recovery blocked.
Bound: only sessions state.json records for this run are added, and only to the recovery prompt's classing, so a path no recorded session wrote stays "Not written by this run".

### D4. `recover-batch` exits non-zero when the batch is not done

A terminal digest whose status is `stuck` or `dead` (either `recovery_retry` value) exits non-zero, after the terminal state's fabric commit.
The envelope stays the `ok: true` digest with every field it has today, so `noteRefusals` writes no refusal note and the call is not a refusal under `PATTERN-refusal-way-forward`: it answers about a batch that did not finish.
A `done` digest and a `running` snapshot from an elapsed `--wait` exit 0; refusals, `batch_failed` among them, keep their current non-zero exits.
Master's failure ladder gains one line: the exit code follows the batch's status, and Master acts on the envelope's `status`, `recovery_retry` and refusal flags as before.
Rationale: an operator or script reading the exit code saw success while the batch had ended terminal and not done (#522).

### D5. The recovery prompt names its allowed gates

The recovery prefix states that every step of the card gates and the batch gate it renders is allowed, naming `lyx gate test` and `lyx loom lint-comments` as examples, and that the ban covers mutating `lyx fabric` verbs only.
It states that the recovery strand runs no `tmux`- or `llm`-tagged step and leaves it to the plan-level verify gate, and that this skip is neither a gate failure nor a `FAILED` report.
Rationale: the `quick-recipe` recovery hesitated over both, and the board entry names this split.
Bound: the paragraph widens no permission the strand's settings deny, and names no command they deny.
The fork path is unchanged: forks inherit Master's context and were not confused.

### D6. `create-not-done` says how the target was looked up

quarry resolves unexported names and names declared in `_test.go` files, so no change is made for exports or test files.
Every `create-not-done` and `rename-not-done` new-side detail names the key resolved (the handle's expected glyph), that quarry resolved it against the worktree's files, and quarry's answer: for `not_found` whether the unit or only the member is missing, and for `ambiguous` each candidate's file with the constraint D7 read from it and why the candidates are not disjoint.
Rationale: the refusal must let an operator tell a missing symbol from a resolver limit.

### D7. Members declared once per disjoint build-constraint set resolve

For Go glyphs, planglyph counts an `ambiguous` answer as resolved when every candidate sits in a distinct `.go` file of the glyph unit's directory (the unit side through `glyph.UnitPath`, the candidate side from quarry's `file`), all candidates are test files or all are not, and the files' build constraints are pairwise mutually exclusive.
A file's constraint is its `//go:build` line, or its `// +build` lines when it has none, together with its `_GOOS`, `_GOARCH` or `_GOOS_GOARCH` filename suffix; a file with neither, or whose line `go/build/constraint` cannot parse, is unconstrained and overlaps every file.
Two constraints are exclusive when no tag assignment satisfies both, with GOOS values mutually exclusive, GOARCH values mutually exclusive and `unix` true exactly for the Unix GOOS values; the known GOOS and GOARCH values are a fixed list owned by planglyph.
Every site that reads `ambiguous` as "not resolved" for a positive verdict uses the rule: the Plan-time status policy over every non-Create glyph target (Edit, Uses, Delete), DoneChecks' Create and Rename new-side checks, and handle binding, which binds a disjoint-ambiguous answer as it binds a multipart one, every candidate a declaration of the one member.
Sites that read `ambiguous` as "a declaration still exists" keep doing so: `delete-not-done`, `rename-not-done`'s old side and `create-already-exists` stay blocking on a disjoint-ambiguous answer.
A non-disjoint `ambiguous` answer stays blocking at Plan time as `glyph-ambiguous`, naming each candidate's constraint, for every target that exists when the plan is validated; a Create target answers `not_found` then, so its build-constraint shape is checked only at record-batch.
Rationale: `tmuxAllowed` in `allow_tmux.go` and `allow_none.go` and `StartTime` in `internal/proc` are one member to the compiler in any one build, so a Create, Edit or Delete of them must be able to pass.
Bound: only an answer whose every candidate is constraint-exclusive with every other passes, so no build sees two declarations; a mix of test and non-test files, an unconstrained file or another directory keeps it ambiguous, and an unknown GOOS or GOARCH reads as an ordinary tag, which can only make files less exclusive.

## Constraints

- `PATTERN-shell-mechanics-seam` widens per D1 in the same commit; `internal/shell` keeps importing the standard library alone.
- `PATTERN-shuttle-provider-seam`: D1 reads `BackgroundTask.Label`, which the provider fills; shuttle core learns no provider text.
- `PATTERN-completion-signal`: D1 changes only which turn ends hold; a negative "did this run finish" answer still consults `allOutputFilesExist` first.
- `PATTERN-no-denied-recovery`: D5 names only allowed commands.
- `PATTERN-glyph-conversion-chokepoint`: D7's unit directory comes from `glyph.UnitPath`.
- `PATTERN-sole-parsers`: D7 reads constraints with the standard library's `go/build/constraint`.
- `PATTERN-test-speed`: unit tier for D1's splitter and match, D2's text, D6's detail and D7's rule; git tier only where a real worktree is needed (D3).
- `pane-inject-safety` is running and rewrites shuttle's verified send, through which D2's longer line goes, and edits `internal/shuttleengine` and its `doc.go`; whichever lands second rebases onto the other.

## Auto-mode assumptions

No operator answered.
Each Decision above is an auto-pick; D5's `tmux`/`llm` skip follows the board entry's wording without the writer confirming why a recovery strand cannot run a `tmux`-tier `lyx gate test`.

## Open risks

- D5: if the plan's `## verify:` runs no `tmux` or `llm` tier, a recovery's tagged step is checked by no one before the PR gate; `weaver` would carry the same skip onto its normal path.
- D6: `cmd/testtiming#rusageStats` (#511) sits in one unconstrained file at base, so its `create-not-done` had a cause this record does not identify; D6's detail is what tells the cause apart next time.
- D3: a recovery session whose id was never recorded (a spawn that crashed before its state commit) stays unattributed, as today.

## Acceptance criteria

- A gated Master run whose first fresh turn end leaves only a background `lyx gate test --tags integration` shell outstanding, with its output files present, does not finish; it finishes after the shell's completion turn, and the same holds after an attach.
- `cd <dir> && lyx gate test ./x`, a newline-separated `cd <dir>` then `lyx gate test ./x`, and `lyx gate test ./x 2>&1 | tail -5` match the `lyx gate ` prefix; `echo "a; lyx gate test"` does not.
- The audit classifier's existing segment cases pass unchanged on the moved splitter.
- A shell that matches no awaited prefix still finishes the first fresh gated arrival at once and is still reported in `Result.EndedShells` and the friction note.
- A failed webster verify gate, on a fresh and on an attached run, sends Master the webster line naming the findings file and the verify-fix prompt file; a gated run of another caller still receives `gateRepromptText`'s line.
- A second recovery spawned after a first recovery left uncommitted files lists them under "Written by this run", including after a Webster restart reclaimed the first strand; a file no recorded session wrote stays under "Not written by this run".
- A recorded recovery session with no transcript yields no writes and no error.
- `recover-batch` exits non-zero with the `ok: true` digest envelope for a terminal `stuck` and for `dead` with either `recovery_retry` value, writes no refusal note, and exits 0 for `done` and for a `running` snapshot.
- The rendered recovery prompt names the card and batch gates as allowed and states the `tmux`/`llm` skip.
- A `create-not-done` detail for a `not_found` target says whether the unit or the member is missing; for an `ambiguous` target it lists each candidate file and its constraint.
- A Create of a member declared in two files with `//go:build tmux || llm` and `//go:build !tmux && !llm` passes DoneChecks and handle binding.
- An Edit of a member declared in `_linux.go` and `_windows.go` files passes the Plan-time status policy and the record-batch scope guard when the card edits both files; a Delete of it passes Plan time and fails `delete-not-done` until both declarations are gone.
- Two `_linux_test.go` and `_windows_test.go` declarations resolve; an `ambiguous` answer with an unconstrained candidate, an unparsable constraint line, a test and a non-test candidate, or overlapping constraints (`linux` and `unix`) stays blocking as `glyph-ambiguous` at Plan time and `create-not-done` at record-batch.
- A Create target that resolves disjoint-ambiguous before the card runs still raises `create-already-exists`.

## Notes for the plan writer

- This is a bench pair: the design faces the code at base `800a0e892`.
- `expiredTurnEnd` and `awaitedLabel` in `internal/shuttleengine/wait.go` decide D1's hold, and the corpus case `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running` records a held turn end; the prefix list is wired in `internal/websterengine/runlevel.go`.
- `gateRepromptText` in `internal/shuttleengine/gate.go` is D2's default; `renderVerifyGateFindings` is in `internal/websterengine/verifygatereport.go`.
- `recordedSessions` and `loadRunWrites` in `internal/websterengine/runwrites.go` feed both the recovery prompt and reset; D3 changes only the former's input.
- `contracts/stencils/claims_test.go` pins the recovery prefix's wording.
- `internal/planglyph/deleteorder.go` reads `StatusAmbiguous` beside `StatusNotFound`; check it against D7's split between positive and still-exists readings.

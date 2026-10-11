# Webster stability: gate shells, Master self-fix, recovery, Create checks

## Goal

Close six Webster faults that stop or mislead runs outside the run-exit audit:
a gated run that finishes `done` while a background gate shell is still running,
Master fixing a verify failure itself,
a second recovery that treats the first recovery's files as foreign,
`recover-batch` exiting 0 over a batch that did not finish,
a recovery prompt that does not say which gates it may run,
and Create, Edit and Delete checks that can never pass for a member declared once per build-constraint set or once per GOOS.

## Scope

In scope:

- `internal/shuttleengine`: the gated waiting-turn-end rule, and an optional per-entry re-prompt line on `GateEntry`.
- `internal/websterengine`: Master's spawn spec, the verify gate's re-prompt and findings text, recording recovery sessions, and run write evidence.
- `internal/webstercli`: `recover-batch`'s exit code and `Long` help.
- `contracts/stencils/webster/`: the Master template's recover-batch rungs and gate-failure section, and the implementer body that both forks and recovery strands read.
- `internal/planglyph`: one partitioned-member classifier over `ambiguous` answers, used by every status decision, and richer done-check finding details.
- Module docs (`doc.go`) of every package above, `contracts/specs/loom-plan-spec.md`, and any refusal-spec row whose wording changes.

Out of scope:

- quarry itself (`github.com/Knatte18/quarry`): it is a separate repository; quarry keeps answering `ambiguous`.
- Ungated shuttle runs' turn-end rule.
- Letting a Create handle reuse a name whose every declaration the same card deletes (#507's first option): partitioned Edit and Delete make it unnecessary.
- Reviving a dead batch through `lyx loom goto` (#511 point 3).
- A deny hook that blocks Master's own edits.

## Decisions

### D1. A gated run's waiting turn end never counts while any background task is outstanding

**Decision:** In `shuttleengine`, a gated run's turn end that leaves any background task outstanding, a shell of either signal or a fork, is never an arrival, with or without every output file present.
The shortcut that counted it Done at once when the run was started fresh, no gated arrival had reached the gate yet and only unawaited shells were outstanding is removed, together with the state that only it reads.
The run waits for the task's completion notification, which starts the agent's next turn; that turn's end is the arrival.
Ungated runs keep today's rule: every output file present finishes Done whatever is outstanding.
`Spec.AwaitedShellPrefixes`, whose only caller is Master's spawn, becomes redundant for gated runs; remove the field and `masterAwaitedShellPrefix` if nothing else reads them, else leave them inert.
`BackgroundShellWaitMin` stays a logging and wait-marker threshold.

**Bound:** the hold is bounded by the run's own deadline (`Spec.Timeout`) and the liveness check, never extended.
A shell that never reports holds the run until its deadline, which finalizes through the existing deadline path and `PATTERN-completion-signal`: with every output file present the run ends `OutcomeDone`, the gate is evaluated once at finalize, and the shell lands in `Result.EndedShells`, so the existing warning, friction note and summary section say its result was never read.

**Rationale:** the shells in #505, #509 and #517 were `lyx gate test` runs, and every agent is told to run `lyx gate test` as a background Bash call, so any gated producer can hit the same fault, not only Master.
A run whose result the agent started and has not read is unfinished work; one rule for every outstanding task is simpler than a label prefix list that a `cd … &&` or an env prefix defeats.
lyx cannot stop a shell itself (the provider reports no pid), so holding is the only way to make the result read.

### D2. The verify gate's re-prompt and findings lead with the fixer-fork rule

**Decision:** `shuttleengine.GateEntry` gains an optional one-line re-prompt renderer, given the findings path; an entry without one keeps today's default line.
Webster's `verify` entry sets it: the line names the findings file, tells Master to spawn exactly one fixer fork with the rendered verify-fix prompt file, end its turn, and never edit, fix or commit anything itself.
`renderVerifyGateFindings` opens with the same way forward before the findings, so the text Master reads at the point of failure carries the rule.
The Master template's gate-failure section stays the full statement; the re-prompt and findings point back at it in words, not a paraphrase of the fork prompt.

**Rationale:** the default re-prompt reads "fix every finding", which tells Master the opposite of its stencil at the moment it acts.
Putting the rule in the message Master reads makes the right path the easy one without a new verb.

### D3. Recovery sessions are recorded on the batch and count as the run's own writes

**Decision:** Every recovery strand spawn records its provider session id on the batch's state record, appended to a list so every spawn of the batch (counted, uncounted amendment re-runs, retries) is kept, and persisted with the state fabric-commit that the spawn already makes.
`loadRunWrites` audits each recorded recovery session beside the Master session and the batch sessions, and carries recovery writes as their own category, never as Master parent writes, so the run-exit audit's parent-write check is unchanged.
Every consumer of the run's write evidence counts recovery writes as the run's own:

- the recovery prompt's uncommitted-paths block lists an earlier recovery's uncommitted files under "Written by this run";
- `reset`'s own-path set includes tracked paths a recovery of this run wrote;
- contract-file evidence counts recovery writes beside fork writes, and its messages name the last writer as a fork or a recovery session.

**Bound:** only successful writes, inside the worktree, by a session this run's state records count; a recovery whose session id was never recorded (a crash between spawn and the state persist, or an unreadable transcript) contributes no evidence, so its files stay under "Not written by this run" as today.
The reset widening lets `reset` discard a tracked path that a recorded recovery wrote; destruction still goes through fabric's chokepoint and its existing checks.

**Rationale:** #522's second recovery blocked because the first recovery's session was never audited; attribution by batch through recorded sessions fixes it without discarding the first attempt's work.

### D4. `recover-batch` exits non-zero unless the batch is done

**Decision:** A terminal `recover-batch` digest whose status is not `done` (`stuck` or `dead`, any `dead_reason`, with or without `recovery_retry`) is an error envelope (`ok: false`) with exit 1, carrying every field the success envelope carries today (digest fields, `recoveries`, `recovery_retry`, `warnings`) and a message naming the way forward from the Master ladder: a second `recover-batch` when `recovery_retry` is true, otherwise the run ends stuck.
The fabric commit of the terminal state still happens before the envelope.
`running` snapshots and `done` digests keep exit 0.
The Master template's recover-batch rungs say the call exits non-zero for `stuck` and `dead` and that the rungs key on the envelope's fields, never on the exit code.
`Long` help states the new exit rule and stays within the CLI form test's length cap.

**Rationale:** the exit code then says whether the batch finished, so neither Master nor an operator reads a failed batch as success; Master's decisions already key on `status` and `recovery_retry`.

### D5. The implementer body names the gates every implementer may run

**Decision:** The implementer body (read by batch forks and recovery strands alike) states, beside the sandbox rule:

- the card gate steps, the batch gate, `lyx loom lint-comments` and `lyx quarry resolve` are always allowed;
- the fabric ban covers mutating `lyx fabric` verbs only;
- a `tmux`- or `llm`-tagged test that the card's change touches and no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, and its run is left to the plan-level verify.

A gate step the card names runs as written, through `lyx gate test` where it names a tag.

**Rationale:** the #522 recovery could not tell whether the gates fell under the fabric ban.
Compiling an unnamed tagged test catches build breaks cheaply; running it is the plan-level verify's job, which the plan rubric already requires to cover build-tagged tests.

### D6. One partitioned-member classifier in `planglyph`

**Decision:** `planglyph` gains one classifier over an `ambiguous` quarry answer: the member is *partitioned* when no single build environment selects two of its candidate files.
Each candidate file is evaluated with `go/build.Context.MatchFile` over every environment in a pinned set: every GOOS/GOARCH pair in a pinned table, cgo on and off, and every true/false assignment of the free tags the candidates' constraints mention.
Free tags include release tags (`go1.N`), compiler tags and `goexperiment.*`: nothing is held true because of the toolchain lyx was built with.
The classifier returns the per-candidate constraint text (the `//go:build` line, or the filename GOOS/GOARCH suffix) for findings to quote.
Every status decision in `planglyph` that reads quarry's answer routes through it: a partitioned answer counts as resolved wherever `found` or `multipart` counts (Edit, Delete, `Uses:`, the Create done-check, a Rename's new side, delete ordering, scope guard).

Exceptions:

- A Create target that resolves partitioned at Plan time is still `create-already-exists`, as ambiguous is today.
- A Rename's old side accepts only `found` and `multipart`, because the new declaration is derived from one signature; its finding names the way forward: a Delete of the old member plus a Create of the new handle, or an Edit of each file's body.
- A Delete's done-check still treats any non-`not_found` answer as still present.

**Bound:** the classifier only turns `ambiguous` into resolved; it never changes `found`, `multipart` or `not_found`.
More free tags than a small fixed cap (four), an unreadable candidate file, an unparsable constraint, or a candidate no environment of the set selects leaves the answer `ambiguous`, so the fallback is today's behaviour.
A genuinely ambiguous Edit or Delete target stays blocked at Plan time by `glyph-ambiguous`.

**Rationale:** quarry ignores build constraints, so `tmuxAllowed` (`allow_tmux.go`/`allow_none.go`), `rusageStats` (`rusage_linux.go`/`rusage_other.go`) and per-GOOS `StartTime`/`runChild` resolve `ambiguous` and no Create or Edit of them could pass; quarry is out of this worktree, so lyx classifies.
Pinning the environments makes the verdict the same on every host.

### D7. Done-check findings say how the target was looked up

**Decision:** `create-not-done`, `delete-not-done` and `rename-not-done` details name the resolve key looked up, quarry's status, the unit status for `not_found`, and for `ambiguous` each candidate's file and build constraint with the classifier's verdict.
`glyph-ambiguous` at Plan time quotes the same candidate file and constraint.

**Rationale:** #511 could not tell a missing symbol from a resolver limit; with D6 the only ambiguity that can first surface at record-batch is one the batch's own Create introduced, and the detail shows it.
Unexported identifiers and test-file members already resolve `found` in quarry v0.2.1, so no resolver change is needed for them.

## Constraints

- `PATTERN-completion-signal`: D1's deadline exit consults `allOutputFilesExist` before any negative verdict; a held shell never turns a satisfied file contract into a failure.
- `PATTERN-shuttle-provider-seam`: D1 and D2 stay provider-neutral in `shuttleengine`; no Claude specifics move out of `claudeengine`.
- `PATTERN-daemon-wakeups`: D1's longer waits reuse the existing poll loop and its wake bound.
- `PATTERN-told-geometry`: `planglyph` reads candidate files under the root it is handed; it imports no `lyxcwd`.
- `PATTERN-fabric-destruction-chokepoint`: D3's reset widening changes only which paths count as own; deletion stays in `fabricengine/destroy.go`.
- `PATTERN-shuttle-stop`: recovery strands keep being stopped only through shuttle's stop verb.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D4's error envelope and D6's Rename finding name the way forward, never a denied command; any refusal-spec row whose wording changes lands in the same commit with a test reaching it.
- `PATTERN-friction-capture`: D1 keeps the `webster-background-shell` note on every outcome.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-producer-pointer-rule`: stencil edits keep the rendered directives and point at the Master template's sections rather than paraphrasing them.
- `PATTERN-comment-line-breaks` and the repo's semantic-line-break rule for every edited comment and `.md` file.
- `PATTERN-test-economy`, `PATTERN-test-speed`: extend existing table tests where one covers the surface; new tests sit in the lowest tier their substrate needs.
- CLAUDE.md "Docs land in the same commit": every changed package's `doc.go` and `contracts/specs/loom-plan-spec.md` change with the code.
- CLAUDE.md "Test budget": a card whose new top-level tests push `internal/shuttleengine`, `internal/websterengine`, `internal/webstercli` or `internal/planglyph` past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- The `recover-batch` `Long` help stays within the CLI form test's length cap.

## Auto-mode assumptions

- D1 applies to every gated run, not only Master, because the background `lyx gate test` instruction reaches every agent.
- D1 leaves ungated runs alone; no reported fault involves one.
- D2 uses a re-prompt line plus findings text rather than a new verb, because the fixer is an in-session fork no verb can spawn.
- D3 keeps the first recovery's work rather than resetting to a clean tree and offering its diff.
- D4 makes every non-done terminal digest exit non-zero, including `dead` with `recovery_retry: true`.
- D5 compiles, never runs, unnamed `tmux`- and `llm`-tagged tests, following the board entry's wording.
- D6 classifies in lyx and leaves quarry unchanged; the free-tag cap is four.
- #511's unexported-identifier hypothesis is false for quarry v0.2.1 (probed: `internal/shuttleengine#awaitedLabel` and a `_test.go` member resolve `found`); both #511 targets were build-constraint duplicates, so D6 and D7 cover #511.

## Open risks

- D1: a gated agent that leaves a shell that never ends (a watcher, a hung test) now holds its run until the run deadline where it finished at once before.
- D1: a shell whose completion notification shuttle never recognizes holds the run until its deadline.
- D2 and D5: stencil and re-prompt wording can only be judged by a live run; no `llm`-tier test is planned for them.
- D4: an operator script that read `recover-batch`'s exit 0 as "terminal digest received" now sees exit 1 for `stuck` and `dead`.
- D5: a plan whose `## verify:` does not run the tmux tier leaves an unnamed tmux-tagged test unrun until Publish.
- D6: the pinned GOOS/GOARCH table can lag a new Go release; a candidate constrained to a new platform alone is then selected by no environment and its member stays `ambiguous` until the table is extended.

## Acceptance criteria

- A gated run whose turn end leaves a background shell outstanding with every output file present does not finish; it finishes at the turn end after the shell's completion notification, through the gate. Covered for a fresh run with no prior arrival, the case the shortcut used to count at once.
- A gated run held on a shell past its deadline ends `OutcomeDone` with the shell in `Result.EndedShells`, and Webster's warning, friction note and summary section name it as unread.
- An ungated run with every output file present still finishes Done at a turn end with a shell outstanding.
- A failed verify-gate evaluation re-prompts Master with the webster line naming the fixer fork and forbidding Master's own edits, and the findings text opens with the same way forward; a gate entry without a renderer keeps the default line.
- A second recovery of a batch whose first recovery left uncommitted writes lists those paths under "Written by this run"; paths no recorded session wrote stay under "Not written by this run".
- A recovery session's writes never appear as Master parent writes in the run-exit audit.
- `reset`'s own-path set includes a tracked path a recorded recovery wrote.
- `recover-batch` exits non-zero with `ok: false` and the full digest fields for a terminal `stuck` and for a terminal `dead` with and without `recovery_retry`; exit 0 for `done` and `running`.
- The rendered implementer body (fork and recovery) names the card gate, batch gate, `lyx loom lint-comments` and `lyx quarry resolve` as always allowed, limits the fabric ban to mutating `lyx fabric` verbs, and gives the compile-only `lyx gate test --tags <tag> … -- -run '^$'` form for unnamed tagged tests.
- The classifier reports partitioned for: a member declared in a `//go:build tmux || llm` file and a `//go:build !tmux && !llm` file; a member declared in `_linux.go` and `_windows.go` files; a member split by `//go:build linux` and `//go:build !linux`.
- The classifier reports not partitioned for: two unconstrained declarations; overlapping constraints (`linux` and `unix`); more than four free tags; an unreadable candidate file; a candidate no environment selects.
- A partitioned member passes the Create done-check, an Edit and a Delete target check at Plan time, `Uses:`, and a Rename's new side; it fails a Rename's old side with a finding naming the way forward, and a Plan-time Create of it is `create-already-exists`.
- A `create-not-done` finding names the resolve key, quarry's status, the unit status for `not_found`, and each ambiguous candidate's file and constraint.

## Notes for the plan writer

- The shortcut lives in `shuttleengine`'s `countsAtOnce`/`expiredTurnEnd` (`wait.go`), with `startedFresh` and `gatedArrival` on `Run`; existing tables in `waiting_test.go` exercise it, and the `master-fork-running` corpus case under `claudeengine/testdata/corpus/` describes the older behaviour.
- The default re-prompt is the `Gate findings recorded at %s` line in `shuttleengine/gate.go`; Webster's entry is built in `websterengine/runlevel.go` and its findings in `verifygatereport.go`.
- Write evidence: `websterengine/runwrites.go` (`loadRunWrites`, `recordedSessions`), consumed by `recoverbatch.go`, `reset.go` and `contractevidence.go`.
- `recover-batch`'s terminal envelope is built near the end of `webstercli/recoverbatch.go`.
- Status checks reading quarry answers in `planglyph`: `resolve.go`, `create.go`, `donecheck.go`, `deleteorder.go`, `handle.go`, `scope.go`; `quarrycli/resolve.go` also filters on found/multipart and may need the classifier if its exit decision should agree.
- Real partitioned members in the tree for test fixtures: `internal/proc#StartTime`; `tmuxAllowed` and `rusageStats` were restructured into one declaration as a workaround.

## Interview

Autonomous run: no operator answered, so every turn below is the writer's own exploration and pick, recorded in the Question ledger.

### Turn 1 — task and board

Read board entry `twb-twh-1` and its linked issues #505, #509, #517, #522, #511, #500 and #507 (all closed upstream; the record faces base `800a0e892` per the entry's bench note).
Related entries read: `fork-wait-noise` (gate-running guidance for forks and Master, in Plan-Write), `plan-glyph-tuning` (test members as glyphs, bind trace), `proc-child-lifetime` (per-GOOS lifetime code in `internal/proc`), `kick-start-pack`, and `twb-twh-2` (the twin bench pair, same body).

### Turn 2 — item 1, background shells

At base, `background_shell_wait_min` already expires nothing.
The remaining hole is `countsAtOnce`: a gated run started fresh, with no gated arrival yet and only unawaited shells outstanding, counts a waiting turn end with every output file present as `Done` at once.
Master's only awaited prefix is `lyx webster recover-batch`, so a `lyx gate test` shell let the run finish and died with the strand.
The `master-fork-running` corpus case shows Master's turn-end list carrying a shell a fork left running.
First pick: drop the exception for every gated run, so every shell holds; narrowed to Master's spawn after the lens review (D1).

### Turn 3 — item 2, Master self-fix

`gateRepromptText` sends every gated run "Gate findings recorded at … — read it, fix every finding, and end your turn."
That is the line Master reads at a verify failure, and it contradicts the Master stencil's fixer-fork rule.
Pick: a per-entry re-prompt line on `GateEntry`, set by webster's verify gate (D2).

### Turn 4 — items 3 and 4, recovery

`uncommittedPathsBlock` classes paths by `loadRunWrites`, which audits Master sessions and their forks only; recovery strands are separate sessions no list records.
`recoverSpawn` already reads the spawned run's state through `shuttleengine.FindRun`, which carries the session id.
Pick: record recovery session ids on `BatchState` and audit them (D3).
`recover-batch` returns `output.Ok` for every terminal digest, so stuck and dead exit 0; pick a failure envelope with the same fields (D4).
The recovery prefix says nothing about which gates are allowed beside "never run `lyx webster`" and the body's fabric rule; tmux-tagged steps come from a card's own `verify:` line, and the plan template leaves the tmux tier to Publish.
Pick: one paragraph in the recovery prefix (D5).

### Turn 5 — items 5 and 6, done-checks and build sets

Live `lyx quarry resolve` checks: an unexported constant resolves found; a test-file constant of an external test package resolves under `<unit>_test`; `internal/proc#StartTime` resolves ambiguous across `proc_linux.go` and `proc_windows.go`.
At base, `cmd/testtiming` holds `rusage_linux.go` and `rusage_other.go`, and `rusageStats` sits in `resources.go`: the #511 target was per-GOOS when it failed, so #511 is the same fault as #500 and #507.
Quarry's `statusForMatches` documents that it does not evaluate build constraints; `DoneChecks` accepts only found or multipart.
`resolveTargets` is planglyph's one call to quarry's resolve, and `lyx quarry resolve` reaches it through `planglyph.Resolve`.
Pick: normalize disjoint-build-set ambiguity to multipart there (D7), and make the done-check text name the answer (D6).

### Turn 6 — lens review

Six lens forks spawned before the files were written, and all six returned.
Duplicates across lenses are merged; each line names its lens or lenses and its disposition.

- scenarios (BLOCKING): auditing a recovery session with no transcript fails `loadRunWrites`, so every later `recover-batch` and `reset` wedges. Fixed: D3 skips such a session with a Warn log; acceptance case added.
- scenarios: a unit collision across two directories passes D7's conditions. Fixed: D7 requires one directory; acceptance case added.
- scenarios: a transcript-signal shell whose completion is never seen now runs Master to its timeout, and no criterion pinned shell-completes-then-Done. Fixed: Open risk and acceptance case.
- scenarios: the session list's fate under `reset --to batch-start` was unstated. Fixed: D3 says it survives.
- scenarios: D6 merged overlap and unreadable constraint. Fixed: D6 names the failed condition.
- scenarios: legacy `// +build` and `//go:build ignore` rows missing. Fixed: D7 text and acceptance rows.
- scenarios: a paused or stuck Master outcome now waits on shells. Fixed: Open risk.
- goal-scope, board-fit: D1 changed every gated run, loom writers and the parent-review wait included, beyond the entry's Master-only fault. Fixed: D1 is now a `Spec` opt-in Master sets; Scope Out names other gated runs.
- goal-scope: #522's exit-0 path was unproven. Fixed: D4's rationale shows only stuck and dead return `output.Ok` at base.
- goal-scope: D5 forbade compile-checking tmux steps. Fixed: clause dropped.
- goal-scope: D7's Plan-time requirement was met only implicitly. Fixed: one sentence on `Create` handles and the shared answer.
- goal-scope, pattern-fit: `quarrycli` and `webstercli` docs and help missing. Fixed: Constraints docs bullet.
- goal-scope: #511's goto halt may still mislead. Rejected: that halt text is not at base; `begin-batch`'s report-present refusal there names `reset --to batch-start`.
- goal-scope, leanness: the acceptance bullet listing tests to update named work, not behavior. Fixed: cut.
- premises: D3's two readers of the write lists were unnamed. Fixed: D3 names `writtenWorktreePaths` and `PlanReset`'s own paths.
- premises: contract-file evidence reads Master and forks, not Master only. Fixed.
- premises: D2 pointed at `NewVerifyGate`, which returns a closure. Fixed: the entry is set where Run builds Master's gate spec.
- premises: the done-only background-shell summary section becomes unreachable. Fixed: D1 removes it.
- pattern-fit: D4's `ok: false` envelope writes a friction note. Fixed: D4 and Constraints say so; acceptance covers it.
- pattern-fit: D7's platform lists had no source. Fixed: a static list pinned by a test against `go tool dist list`.
- pattern-fit: D2's Go text must point at the stencil, not restate it. Fixed: D2's line carries only the pointer and the path.
- board-fit: `plan-glyph-tuning`, `shuttle-startup-ready` and `parent-review-wait` edit the same files unnamed. Fixed: Notes line of parallel editors.
- board-fit: D6's `not_found` text misleads for a member in the sibling `_test` unit. Fixed: D6 names the sibling-unit match.
- board-fit: `proc-child-lifetime` needs D7. Fixed: Notes line.
- board-fit: `fork-wait-noise` edits the body recovery strands share. Fixed: Notes names the body.
- board-fit: `weaver` reuses the recovery pieces. Rejected: no conflict, and the record's decisions already state the rules it would read.
- board-fit: the `kick-start-pack` and `plan-glyph-tuning` entries contradict each other. Rejected: outside this record.
- leanness: Goal restated Scope and Decisions. Fixed: Goal is two sentences.
- leanness: Auto-mode assumptions restated picks. Fixed: each line states why it was assumed, and the scope repeat is gone.
- leanness: D5 repeated the no-denied-recovery constraint. Fixed: cut from D5.
- leanness: Scope item 7 overlapped Constraints. Fixed: the docs list lives in Constraints only.
- leanness: the test-budget constraint restated CLAUDE.md. Fixed: cut.
- leanness: the `fork-wait-noise` note repeated Scope Out. Fixed: Notes keeps only the parallel-edit fact.

## Rejected alternatives

- **D1, a label prefix for gate shells** (`lyx gate test` added to `AwaitedShellPrefixes`): misses `cd …` and other spellings and every other verification shell, and still needs label matching rules.
- **D1, every shell holds every gated run's turn end**: the first draft's pick; it changes loom writers and the parent-review wait, which the entry never reported, so Master opts in instead.
- **D1, lyx stops the shell and records the discarded result**: the provider reports no pid, so lyx cannot stop a shell; holding the turn end needs no kill.
- **D2, a verb that spawns the verify fixer**: the fixer is an in-session Agent-tool fork, which no CLI verb can spawn.
- **D2, a line at the top of the findings file**: the fixer fork reads the same failure record, and an instruction not to fix would contradict its job.
- **D3, attribution by batch only**: other batches are terminal and committed, so the narrower unit gains nothing and diverges from how fork writes are attributed.
- **D3, a second recovery from a clean tree with the first attempt's diff as reference**: needs a destructive reset of uncommitted work before a strand that may reuse it.
- **D7, change quarry to honour build constraints**: a cross-repository change against quarry's documented design, which keeps the engine platform-free.
- **D7, accept disjoint ambiguity only in `DoneChecks`**: Plan time and record-batch would read different answers, and `Edit`/`Delete` by glyph (#507) would stay blocked.
- **D7, a Create handle reusing a name the same card deletes everywhere** (#507's first option): a plan-grammar change the second option makes unnecessary.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): hold every shell or only gate shells, and for which runs? Every shell, for Master's spawn only (D1); revised from every gated run after the lens review.
- Q2 (auto-pick): message text or a verb for Master's verify-fix path? A per-entry re-prompt line (D2).
- Q3 (auto-pick): attribution unit for recovery writes? The run, with recovery sessions recorded (D3).
- Q4 (auto-pick): does a dead digest with `recovery_retry: true` exit non-zero? Yes, the batch is not done; the message names the retry (D4).
- Q5 (auto-pick): which tagged steps does the recovery prompt leave to verify? `tmux` only, as the board entry names (D5).
- Q6 (resolved by reading): does the Create check fail to resolve unexported or test-file members? No; #511's target was per-GOOS (D6).
- Q7 (auto-pick): where does build-set disjointness live? planglyph's resolve chokepoint, reaching `lyx quarry resolve` too (D7).
- Q8 (auto-pick): #507's Create-reuse rule and #511's goto revival? Out of scope, per the decision record's Scope.

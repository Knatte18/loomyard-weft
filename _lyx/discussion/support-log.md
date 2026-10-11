# Support log — Webster stability: gate shells, Master self-fix, recovery, Create checks

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.
The record faces base `800a0e892`, read with `git show 800a0e892:<path>`.

1. Board read: task `twb-twp-1`, items 1–6 with issues #505, #509, #517, #522, #511, #500, #507.
   Adjacent entries read: `fork-wait-noise` (how forks wait for gates), `proc-child-lifetime`, `audit-reader-globs`, `verify-watchdog`, `discussion-premise-cites`, `gate-checklist-pane`, `tokencount-crash-gap`.
   Only `fork-wait-noise` overlaps: it changes how forks run gate shells and the Master template's gate wording, and this record leaves that to it.
2. Item 1, gate shells: at base no shell expires any more (`internal/shuttleengine/doc.go`, `background_shell_wait_min` is display only), and `Result.EndedShells` already feeds a friction note and a run warning.
   The gap left: a gated run's waiting turn end with both output files present finishes Done at once when every outstanding task is an unawaited shell, and Master awaits only `lyx webster recover-batch`.
   Labels are the shell's command (`claudeengine/background.go`), so a `cd <pair> && lyx gate test …` label misses a start-of-label prefix.
   Pick: await `lyx gate test` and match prefixes in command position.
3. Item 2, Master self-fix: the verify gate's re-prompt is shuttle's generic `gateRepromptText`, "read it, fix every finding, and end your turn", which contradicts Master's stencil.
   Pick: a per-entry re-prompt text; webster's names the fixer fork.
4. Item 3, recovery attribution: `loadRunWrites` audits Master sessions and their forks only; a recovery strand is a separate top-level session no state field records, so its writes are never evidence.
   Pick: record each recovery session id and audit it as a third write class, kept out of the contract-file evidence.
   `recover-batch` returns `output.Ok` for every terminal digest, stuck and dead included; pick: non-zero for any non-done terminal digest.
5. Item 4, recovery gates: the fabric audit's spelling rule matches only `lyx fabric|weft|warp` (`fabricengine/refscanner.go`), and `lyx gate test` strips the strand-name variable, so the gates were always allowed; the doubt came from the body's sandbox rule.
   Card gates carry no tmux tag (`cardgate.go`); a tmux-tagged step can come only from a card's own `verify:` line, which the settings deny as a raw `go test`.
   Pick: state the allowed gates in the shared body and leave tmux- and llm-tier `verify:` steps to the verify gate.
6. Item 5, Create on unexported targets: checked with `lyx quarry resolve` (quarry v0.2.1 at base and now): `internal/websterengine#loadRunWrites` and `internal/websterengine#fakeClock` (same-package `_test.go`) resolve `found`; `internal/shuttleengine_test#…` resolves external-test members; `internal/proc#StartTime` resolves ambiguous.
   So the resolver handles unexported and test members; #511's two targets most likely failed as per-build-variant declarations, the same as #500.
   Pick: make the detail name the status, candidates and the `_test` unit hint.
7. Item 6, build variants: quarry is an external module and ignores build constraints; DoneChecks read `ambiguous` as not created.
   Pick: one lyx-side build-variant predicate read by every planglyph status rule.
   #507's per-GOOS Edit and Delete are covered by the same predicate; its recreate-a-deleted-name option is out.
   #511's unexported targets most likely failed as build-variant ambiguity, the same shape as #500.
8. Lens review: six forks, all returned. One line per finding, duplicates merged.
   - premises BLOCKING, missing recovery transcript fails the audit rather than falling back: fixed, Decision 4 states the recovery-class leniency and its bound; the open risk is removed.
   - premises MEDIUM, the reset reads `ownTrackedPaths`, not `writtenWorktreePaths`: fixed, Decision 4 names both readers.
   - premises MEDIUM, `tmuxAllowed` is not ambiguous at base: fixed, Decision 7 cites `StartTime` and calls #500's shape historical.
   - premises LOW, "a Go answer" has no source: fixed, `plan.GlyphLanguage()`.
   - premises LOW, `statusFindings` already tells member from unit: fixed, Decision 8 reuses its wording.
   - goal-scope MEDIUM, Decision 5 vs the board's "failed": fixed as an auto-mode assumption; the failed path already exits non-zero.
   - goal-scope MEDIUM, no Plan-time flag for a Create: fixed, listed as a departure under Auto-mode assumptions.
   - goal-scope LOW, "Master runs no gate" is unlicensed: kept, listed as an assumption; it narrows Master to its stencil's role.
   - goal-scope LOW, Publish-vs-Webster verify: fixed as an assumption.
   - goal-scope LOW, Decision 6 reaches forks: fixed in Scope.
   - goal-scope LOW, external-test member not fixed: fixed in Scope Out.
   - pattern-fit MEDIUM and board-fit MEDIUM, "left to verify" contradicts `PATTERN-test-speed` and the body's raw-`go` line: fixed, `tmux` steps run through `lyx gate test --tags tmux`, `llm` steps are skipped and named.
   - board-fit MEDIUM, `fork-wait-noise` overlap not ordered: fixed, a Constraint splits ownership.
   - board-fit LOW, quarry pin vs `enclose-kickstart`: fixed, a Constraint.
   - board-fit LOW, `weaver` reuse and `glyph-practice-audit`: rejected, no dependency or conflict to record.
   - pattern-fit LOW, re-prompt policy in Go vs `PATTERN-stencil-ownership`: answered in Constraints, a one-line send around a runtime path like `gateRepromptText`.
   - pattern-fit LOW, refusal-spec row kind: fixed, a terminal failure report.
   - pattern-fit LOW, duplicate command-position matcher: fixed, shuttle exports one matcher that Decision 2 reuses; `refscanner.go` stays fabric's own, since shuttle may not import fabric.
   - pattern-fit LOW, test-budget rule: fixed, a Constraint.
   - scenarios MEDIUM, Master's turn on a gate shell's completion: fixed, Decision 3's stencil rule.
   - scenarios MEDIUM, session id recording and resume paths: fixed in Decision 4 and Acceptance criteria.
   - scenarios MEDIUM, no flag on the stuck/dead envelope: fixed, `recovery_failed: true`.
   - scenarios MEDIUM, binding and `answerSymbols` over a variant set: fixed in Decision 7 and Acceptance criteria.
   - scenarios LOW, `+build`, `ignore`, unparseable, `_test` variants, partial platform cover: fixed in Decision 7 and Open risks.
   - scenarios LOW, `lyx gate test --tags tmux` vs skip conflict: resolved by the Decision 6 change.
   - scenarios LOW, Decision 2's match rule: fixed, it uses Decision 1's matcher.
   - scenarios LOW, attached or resumed run: fixed, one sentence in Decision 1.
   - scenarios LOW, slot-busy counts against the timeout: fixed in Decision 1's bound.
   - scenarios LOW, dirty-worktree re-prompt: fixed, the text covers both causes.
   - leanness LOW: the "no pid" repeats, the Auto-mode rationale clauses, the single-line Constraint, the test-naming clause and Decision 4's closing sentence are cut; the #511 attribution moved here; the Scope In list is shortened to its packages.

## Rejected alternatives

- Stopping a gate shell explicitly at the run's end: the provider reports no pid, and `shuttleengine` documents that lyx kills no shell.
- Awaiting every background shell: a stray long shell would hold every Master run until its timeout; gate runs are the ones whose result matters.
- Webster passing more literal prefixes (`cd … && lyx gate test`) instead of a command-position match: the directory varies per pair and the pipe tail per call.
- A webster verb that spawns the verify fixer: a fork can only be spawned by Master's Agent tool, and a verb would need a cold strand that loses Master's context, a second recovery mechanism.
- Stating the fixer rule only in the stencil again: it already is, and Master followed the conflicting gate message.
- Second recovery from a clean tree with the first attempt's diff as reference: throws away work the prompt already knows how to keep, and adds a diff artifact and a reset step.
- Attributing by shuttle run id or by batch only: the run-wide session audit already exists, and the missing piece was one session class.
- Leaving `tmux`-tier `verify:` steps to the plan-level verify, as the board entry states: `PATTERN-test-speed` has that verify run no `tmux` tier, so the step would go unrun until Publish.
- Failing the audit on a missing recovery transcript, as for Master: one early-stopped recovery would block every later recovery prompt and reset.
- Teaching quarry build constraints: quarry is a separate module outside this repository.
- Accepting any ambiguous Create answer: would pass a real duplicate declaration.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Hold or stop a gate shell at a turn end? Auto-pick: hold (Decision 1), since lyx has no pid. Resolved.
- Q2 Where does the rule against self-fixing reach Master? Auto-pick: in the verify gate's re-prompt text (Decision 3). Resolved.
- Q3 Attribute the first recovery's files, or restart clean? Auto-pick: attribute through recorded recovery sessions (Decision 4). Resolved.
- Q4 Which terminal digests exit non-zero? Auto-pick: stuck and dead; done and the running snapshot stay 0 (Decision 5). Resolved.
- Q5 Does Decision 6's wording go in the recovery prefix or the shared body? Auto-pick: the shared body, since forks meet the same gates and denies. Resolved.
- Q6 Does the resolver miss unexported or test members? Resolved by reading: no, both resolve `found` (Decision 8).
- Q7 Fix build variants in quarry or lyx? Auto-pick: lyx-side predicate (Decision 7). Resolved.
- Q8 Include #507's recreate-a-deleted-name option? Auto-pick: no; its per-GOOS option is covered (Scope). Resolved.
- Q9 Should Master be told to run no gate itself? Auto-pick: yes, one stencil line beside the re-prompt change (Decision 3). Resolved.
- Q10 Do `tmux`-tier `verify:` steps run or go to verify? Auto-pick after lens review: run through `lyx gate test --tags tmux`; `llm` steps are skipped and named (Decision 6). Resolved.
- Q11 Missing recovery transcript: error or no evidence? Auto-pick: no evidence, logged (Decision 4). Resolved.
- Q12 Who launched the gate shells in #505, #509 and #517, Master or a fork? Open: the issues do not say; Decision 1 holds either way.

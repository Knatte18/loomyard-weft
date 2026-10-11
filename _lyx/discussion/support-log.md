# Support log: twb-twc-2

## Interview

The session ran autonomously, so every turn below is a self-answered question, with its pick in the Question ledger.

1. Read board entry `twb-twc-2` and the touching entries: `fork-wait-noise`, `proc-child-lifetime`, `weaver`, `kick-start-pack`, `verify-watchdog`, `discussion-premise-cites`, `audit-reader-globs`.
   Issues #505, #509, #517, #522, #511, #500 and #507 read through `gh`.
   The code was read at base `800a0e892`.
2. Gate shells: at base no shell expires.
   `countsAtOnce` still finishes a fresh gated run's turn end at once when the output files exist and every outstanding task is an unawaited shell.
   Master's only awaited prefix is `lyx webster recover-batch`, and a shell's label is its command text.
   The pick was to hold gate shells by adding an awaited prefix, matched at any segment start.
3. Master self-fix: the verify failure reaches Master as `gateRepromptText`'s "read it, fix every finding, and end your turn", which contradicts the template.
   A verb that spawns the fixer cannot exist, because the fixer is an in-session fork.
   The pick was a caller-supplied re-prompt per gate entry.
4. Recovery attribution: `loadRunWrites` audits only Master session ids, and recovery strands record no session id.
   The pick was to record recovery session ids on the batch and audit them, keeping them out of contract-file evidence.
   Also: a `stuck` or `dead` terminal digest goes through `output.Ok`, so it exits 0.
5. Recovery gates: the sandbox rule bans mutating `lyx fabric` verbs only, and the raw-go deny allows `lyx gate test --tags tmux`.
   `loom-plan-spec.md`'s verify model leaves the tmux tier to `publish_verify`.
   The pick was a recovery-prefix section naming the gates and the tmux and llm skip.
6. Create check: `lyx quarry resolve` answers `found` for `cmd/testtiming#rusageStats` (unexported) and for a `_test.go` member.
   It answers `ambiguous` for `internal/proc#StartTime` (per-GOOS) and, per #500, for the build-tagged `tmuxAllowed`.
   So #511 is #500's fault plus a refusal that hides quarry's answer.
   `statusFindings` blocks every ambiguous Edit, Delete and `Uses:` target at Plan time (#507).
   The pick was one build-split predicate in `planglyph`, read as `multipart`, an informational plan-time flag, and a fuller `create-not-done` detail.
7. Spawned six lens forks before writing the files.
8. Lens review: all six forks returned.
   Findings, merged across lenses:
   - goal-scope + scenarios + premises (BLOCKING/MEDIUM): no evidence that a fork-left gate shell's completion starts a Master turn, so an unbounded hold could idle to the deadline. Fixed: Decision 1 holds up to `gate_shell_hold_min`.
   - scenarios (BLOCKING): at the deadline `classifyDeadlineExpiry` answers `done`, not `timeout`. Fixed: the deadline acceptance case now says `done`, and new Decision 2 records a cut-off gate shell as discarded.
   - goal-scope + scenarios (MEDIUM): when the recovery session id is recorded. Fixed: at spawn, with the spawn's state commit; a reclaimed-mid-work acceptance case added.
   - scenarios (MEDIUM): a recorded recovery transcript that can't be read. Fixed: no evidence and a warning, so its paths read as foreign.
   - scenarios + board-fit (MEDIUM/LOW): other `recover-batch` exit-code consumers. Fixed: the bound states that no Go caller branches on it (the `loomcli` and `burlercli` mentions are comments); re-entrant calls covered.
   - premises (MEDIUM): implied GOOS tags (android, ios, illumos) and `unix` in expressions. Fixed in Decision 10, with an acceptance case.
   - premises (LOW): a `linux.go` prefix is not a suffix. Fixed.
   - scenarios (LOW): `// +build`-only files. Fixed: they read as not build-split.
   - scenarios (LOW): a build-split Edit and the deviation union. Fixed.
   - scenarios (LOW): a bad caller re-prompt. Fixed: a gate error.
   - goal-scope (MEDIUM): Creates can't be flagged at Plan time. Fixed: Decision 11 says so.
   - goal-scope (MEDIUM): the #507 exclusion reason was overstated. Fixed: the recreate case stays refused and is spelled as an Edit.
   - goal-scope (LOW): the `llm` skip goes beyond the board. Fixed: the rationale names the verify model.
   - goal-scope (LOW): a pinned "six faults" count. Fixed.
   - board-fit (MEDIUM): overlap with `fork-wait-noise`'s gate spelling. Fixed: Decision 3 ties the prefix set to it.
   - board-fit (LOW): quarry version. Fixed: the Goal names `v0.2.1`.
   - board-fit (LOW): `weaver` inherits the recovery changes. Fixed: an Open risks line.
   - board-fit (LOW): `glyph-practice-audit`. Rejected: no conflict, since the finding is informational.
   - pattern-fit (MEDIUM): Decision 8 restated the verify model. Fixed: the section points at the spec.
   - pattern-fit (LOW): unit comparison and `PATTERN-glyph-conversion-chokepoint`. Fixed in Constraints.
   - pattern-fit (LOW): the stencil-ownership note for the re-prompt line. Fixed in Constraints.
   - pattern-fit (LOW): the test budget. Fixed in Constraints.
   - pattern-fit (LOW): label-matching tests belong in `shuttleengine`. Fixed.
   - leanness (MEDIUM): a rejected alternative inside Decision 4. Fixed: cut.
   - leanness (MEDIUM): Auto-mode assumptions repeated the Decisions. Fixed: one line each with a pointer.
   - leanness (LOW): Decision 8's investigation narrative, an Open risks line repeating a bound, the directive-markers constraint, the #507 reason, and the Goal list. All fixed, except the Goal list, which is kept as the summary the plan writer scans.
   - premises (LOW): `rusageStats`'s refusal cause is not established. Fixed: Decision 9 no longer claims a cause.
   - premises (LOW): "Master followed the message" is an inference. Rejected: it is the only instruction to self-fix Master received, and the record states it as the board's observation.

## Rejected alternatives

- Stop the gate shell and record its result as discarded: lyx has no pid for a provider shell, and holding gives Master the result.
- An unbounded hold on gate shells: a shell a fork left running may never start a Master turn, so the run would idle to its deadline.
- Demoting a run to stuck when a gate shell is cut off: the plan-level verify at HEAD already passed, so `done` is not a false test verdict.
- Re-reading transcripts during the hold for a fork-left shell's completion: it relies on notification placement nothing has shown.
- Await every background shell, not only gate runs: an unrelated long shell would hold every run to its timeout.
- Match `lyx gate` as a substring anywhere in the label: it matches inside paths and arguments too freely.
- A `lyx webster` verb that spawns the verify fixer: a verb cannot start an in-session fork.
- Restating the rule only in the Master template: the template already says it, and Master followed the re-prompt line instead.
- A second recovery from a clean tree with the first attempt's diff as reference: it discards work the prompt already knows how to keep.
- Adding recovery writes to `RunWrites.Master`: they would then clear contract-file findings.
- Teaching quarry build constraints: it is a separate repository, and the plan-gate and done-check policy are lyx's.
- Letting a Create handle reuse a name its own card deletes in full: not needed once per-GOOS members resolve by glyph.
- A blocking `glyph-build-split`: the plan is coherent, so blocking would recreate #507.
- The tmux and llm skip in the shared implementer body: the board names the recovery prompt, and the fork's behavior was not reported as broken.

## Review rounds

_No rounds yet._

## Question ledger

- Q1, hold or stop a running gate shell? Auto-pick: hold up to `gate_shell_hold_min` (default 20), then record the shell as discarded. Revised after the lens review. Resolved.
- Q2, which shells count as gate shells? Auto-pick: `lyx gate test` at the start of any command segment. Resolved.
- Q3, where does Master read the fixer rule? Auto-pick: a webster-supplied gate re-prompt line, plus the template section's opening. Resolved.
- Q4, attribution by batch or clean tree? Auto-pick: attribution by batch, through recorded recovery session ids. Resolved.
- Q5, which `recover-batch` outcomes exit non-zero? Auto-pick: terminal `stuck` and `dead`, with the digest fields kept. Resolved.
- Q6, which recovery steps are skipped? Auto-pick: only `tmux`- and `llm`-tagged steps, in the recovery prompt only. Resolved.
- Q7, does the Create check miss unexported or test-file members? Answered from the code: no, both resolve `found`. Resolved.
- Q8, quarry or lyx for build constraints? Auto-pick: lyx `planglyph`, failing closed. Resolved.
- Q9, should the plan-time flag block? Auto-pick: informational. Resolved.
- Q10, #507's Create reuse of a deleted name? Auto-pick: out of scope, made unnecessary by Q8. Resolved.

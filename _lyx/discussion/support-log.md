# Support log: Webster stability — gate shells, Master self-fix, recovery, Create checks

## Interview

The session ran autonomously: each turn below is a question the writer posed from the board entry and the base-`800a0e892` code, answered by its own pick (ledger entries Q1–Q12).

1. **Is item 1 still live at base?**
   `background_shell_wait_min` no longer expires anything at base, so #505's "after 10 minutes" path is gone.
   The remaining path is `countsAtOnce`: a fresh gated run's first waiting turn end with both output files present counts at once when every outstanding task is an unawaited shell.
   Master's gate shells are unawaited, so the fault stands. → Decision 1 (Q1, Q2).
2. **Hold or stop the shell?**
   Holding reads the result; stopping only records that it was lost.
   The awaited-prefix mechanism already exists for `recover-batch`. → hold, Master only (Q1, Q2).
3. **Does a prefix match catch the labels seen?**
   The payload label is the command text; `… | tail` matches, `cd <pair> && lyx gate test` does not. → split at command separators (Q3).
4. **Where does Master read a verify failure?**
   In shuttle's generic re-prompt, which says "fix every finding". → per-entry re-prompt override (Q4).
5. **Why did the second recovery see the first one's files as foreign?**
   `loadRunWrites` audits Master sessions and their forks; a recovery strand is its own session, recorded nowhere. → record recovery session ids per batch (Q5, Q6).
6. **Which recover-batch exit was 0?**
   At base the `batch_failed` path already exits non-zero; terminal `stuck` and `dead` digests return `ok`. → non-zero for those two (Q7).
7. **What blocks a recovery's gates?**
   Only `lyx webster` and mutating `lyx fabric` verbs are banned; the tmux tier is `publish_verify`'s per the plan spec. → name both in the recovery prefix (Q8).
8. **Does the Create check resolve unexported and test-file members?**
   Yes: quarry v0.2.1 has no export filter and folds same-package `_test.go` files into the unit; probing resolves `tmuxkit#checkSockets` and `testtiming#TestMain` `found`.
   At base `rusageStats` is one declaration in `resources.go` beside `rusage_linux.go`/`rusage_other.go`, and `tmuxAllowed` was restructured into a `var` plus `init()`: the run's workarounds for build-variant ambiguity.
   `internal/proc#StartTime` still resolves `ambiguous` over `proc_linux.go`/`proc_windows.go`. → Decision 6 and 7 (Q9, Q10).
9. **Where does the build-variant rule live?**
   quarry is another repository; planglyph already owns every status policy. → planglyph (Q9).
10. **What does "flag at Plan time" mean once DoneChecks accept build variants?**
    The status policy uses the same predicate, so a non-variant ambiguous Edit target is refused at Plan time with each file's constraint named; a Create cannot be judged before it exists. → Decision 6 (Q11).
11. **Board fit.**
    `fork-wait-noise` rewrites how forks and Master wait on gates and edits the Master template; this record leaves fork prompts alone and expects a textual merge on the Master template.
    `plan-glyph-tuning` adds a bind trace line at record-batch, which does not overlap Decision 7's detail text.
    `twb-twh-1` is the same entry benched separately. (Q12)
12. **Lens review.**
    All six forks returned reports.
    - board-fit, BLOCKING, Decision 3's line forced one fixer fork and would override `fork-wait-noise`'s no-second-fixer rule: fixed; the line now points at the stencil section instead of restating it.
    - board-fit, LOW, Decision 1 is a backstop once `fork-wait-noise` runs gates in the foreground: fixed in Decision 1's rationale.
    - board-fit, LOW, unnamed overlap with `shuttle-startup-ready`, `parent-review-wait` (`wait.go`, `gate.go`) and `plan-glyph-tuning`'s bind code: fixed in the Notes.
    - board-fit, LOW, a fork sees an `ambiguous` answer for a per-GOOS Edit target: rejected; the ambiguous answer already lists every candidate file, and fork prompts are out of scope.
    - board-fit, LOW, Decision 8 should not restate `run_in_background`: no change needed; it does not.
    - goal-scope, MEDIUM, the exit-0 case is inferred: fixed as an Auto-mode assumption with its evidence.
    - goal-scope, MEDIUM, a held shell's result has no consumer, and why Master runs gates: fixed in Decision 1 (the verify gate is the verdict, the hold keeps the run from killing it); a Master stencil ban rejected, see Rejected alternatives.
    - goal-scope, LOW, Plan-time flagging reinterpretation lived only in the log: fixed as an Auto-mode assumption.
    - goal-scope, LOW, the reset change was a side effect: fixed in Scope In item 3.
    - goal-scope, LOW, external test packages not addressed in Scope: fixed in Scope Out.
    - goal-scope and leanness, LOW, "six faults" tally: fixed.
    - premises, MEDIUM, handle binding already counts a variant set as one glyph: fixed; Decision 6 states the base behaviour, criterion 18 is a regression pin.
    - premises, LOW, `tmuxAllowed` is `found` at base: fixed; cited in #500's original shape beside `internal/proc#StartTime`.
    - premises, LOW, the session id comes from `recoverSpawn`'s existing `FindRun`: fixed.
    - premises, LOW, clearing is not in `reset.go`: fixed; cleared wherever `Recoveries` is.
    - premises and scenarios, LOW, restart between recoveries: checked `RemoveRecoveryStrands` only stops strands; the re-begin carry added; criterion 7 covers both.
    - pattern-fit, MEDIUM, at the deadline a run with both files present finalizes `done`, not `timeout`: fixed; checked `finalize` evaluates the gate at done, so Decision 1's bound and criterion 4 state it.
    - pattern-fit, MEDIUM, Decision 8's raw vet and the deny's steer toward `lyx gate test --tags tmux`: partly fixed (package-scoped vet, named deny); running the step rejected, see Rejected alternatives.
    - pattern-fit, LOW, the new row's class: fixed (correctness halt).
    - pattern-fit, LOW, the `recover-batch` `Long` text: fixed in the docs list.
    - pattern-fit, LOW, Decision 3 duplicating the stencil: fixed by the pointer wording and the pin in Constraints.
    - pattern-fit, LOW, the splitter's home: fixed in Constraints with the reason it stays in `shuttleengine`.
    - scenarios, MEDIUM, the `ok: false` envelope writes a refusal friction note: fixed; stated as wanted, criterion 12.
    - scenarios, MEDIUM, which evidence list recovery writes join: fixed; the non-Master side, criterion 10.
    - scenarios, MEDIUM, no worst-case and resumed-run criteria: fixed, criteria 3 and 4.
    - scenarios, LOW, transcript-only shell: fixed in criterion 2.
    - scenarios, LOW, an invalid override text: fixed in Decision 3.
    - scenarios, LOW, session-id failure and crash window: fixed in Decision 4's bound.
    - scenarios, LOW, Rename old side, three-way set, `_test` suffix: fixed in Decision 6 and criterion 14.
    - scenarios, LOW, llm and other tag spellings: fixed in Decision 8 and criterion 13.
    - leanness, MEDIUM, Auto-mode assumptions re-argued rejected alternatives: fixed; trimmed to real assumptions.
    - leanness, MEDIUM, Scope Out cited Decision 2 for the hook: fixed; the reason lives in Rejected alternatives.
    - leanness, LOW, the #511 narrative in Decision 6: fixed; one clause kept.
    - leanness, LOW, criterion 18 pinned unchanged behaviour: fixed; dropped.
    - leanness, LOW, Constraints bullets that change nothing: fixed; sole-parsers and test-budget dropped, completion-signal kept as a substantive bound.
    - leanness, LOW, the cross-machine open risk restated Decision 4: fixed; dropped.
    - leanness, LOW, the stencil-ownership argument belongs in Constraints: fixed.

## Rejected alternatives

- **Stop the gate shell and record its result discarded** (item 1): loses the result the run needs; holding is bounded by the same deadline.
- **Await every shell at a gated turn end:** a stray long-lived shell (a `tail -f`, a Monitor) would hold Master until `master_timeout_min`.
- **Make `lyx gate test` awaited for every gated run in shuttle:** no other run showed the fault; YAGNI.
- **A `lyx webster` verb that spawns the verify fixer:** forks are Agent-tool subagents of Master's session, so a verb could only spawn a separate strand, a second fixer mechanism beside the fork.
- **A PreToolUse deny on Master's Edit and Write:** Bash writes bypass it, it needs provider-seam plumbing, and the run-exit parent-write audit already catches the write.
- **A Master stencil line forbidding `lyx gate test`:** `fork-wait-noise` is rewriting how agents run gates in the same template, and the hold makes a Master-run gate safe either way.
- **Moving the label split into `internal/shell`:** that package builds command strings; matching an outstanding task's label is shuttle's wait decision.
- **Having the recovery run tmux-tagged steps through `lyx gate test --tags tmux`:** the board asks that they be left to verify, and the plan spec gives the tmux tier's run to `publish_verify`.
- **Restating the rule in the Master stencil only:** the stencil already says it; Master followed the sent line instead.
- **Second recovery from a clean tree with the first attempt's diff as reference** (item 3): discarding uncommitted work needs a destructive step, and a diff is a weaker start than the work in place.
- **Attribute by shuttle run directory instead of session id:** a `done` recovery's run directory is removed, and the audit reads transcripts by session.
- **Teach quarry build constraints:** a cross-repository change and release; planglyph owns the policies.
- **Accept any ambiguous answer whose candidates all carry some constraint:** `linux` and `linux && amd64` compile together, so that would pass a real duplicate.
- **Let a Create reuse a name its own card deletes** (#507's second ask): unnecessary once Edit and Delete work by glyph on build variants; not in the board's expected outcome.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): hold or stop a running gate shell at Master's turn end? → hold via an awaited prefix; reads the result, bounded by `master_timeout_min`.
- Q2 (resolved, auto-pick): awaited for Master only or for every gated run? → Master only; the fault is Master's.
- Q3 (resolved, auto-pick): how to match `cd … && lyx gate test`? → split the label at `&&`, `||`, `;`, `|`; a plain token split.
- Q4 (resolved, auto-pick): how to put the fixer rule where Master reads the failure? → optional per-entry re-prompt text on `GateEntry`, set by webster's verify entry; after the lens review it points at the stencil's gate-failure section instead of restating it.
- Q5 (resolved, auto-pick): attribution by batch or clean restart? → attribution by the batch's recorded recovery sessions.
- Q6 (resolved, auto-pick): what if a recovery transcript is unreadable? → no evidence, logged at Warn, spawn proceeds.
- Q7 (resolved, auto-pick): which recover-batch terminals exit non-zero? → `stuck` and `dead`; `done` and `running` stay 0; digest fields kept on the error envelope.
- Q8 (resolved, auto-pick): what does the recovery do with a tmux- or llm-tagged step? → a package-scoped `go vet -tags <tag>`, leaving the run to the plan-level verify and `publish_verify`.
- Q9 (resolved, auto-pick): fix build-variant ambiguity in quarry or planglyph? → planglyph.
- Q10 (resolved, auto-pick): does the check resolve unexported and test-file identifiers? → yes; the #511 failures were build-variant ambiguity; Decision 7 makes the lookup visible.
- Q11 (resolved, auto-pick): what does Plan-time flagging cover? → the status policy uses the same predicate, and a non-variant `glyph-ambiguous` names each file's constraint.
- Q12 (resolved, auto-pick): board dependencies? → none blocking; `fork-wait-noise` shares the Master template, noted for the plan writer.

# Support log — dbench-n0om

## Interview

Autonomous mode, no advisors: every turn below is a question the writer settled from the board entry, the linked issues and the code.

1. Fault 1 (background gate shell). The code already stops expiring shells and records each shell outstanding at the end (commit 5a8be0274), but a gated turn end with both output files present still finishes when only unawaited shells are outstanding. shuttle cannot stop a shell (no pid), so holding is the only way to observe a result. Picked: add `lyx gate ` to Master's awaited prefixes (D1) and word the existing record as "not observed" (D2).
2. Fault 2 (Master self-fix). Master's stencil already says "Do NOT localize or fix the failure yourself" at the gate-failure section, but shuttle's re-prompt says "read it, fix every finding, and end your turn". Picked: a per-entry re-prompt text, webster's naming the fixer-fork route (D3).
3. Fault 3 (second recovery). `recordedSessions` reads Master's and the batches' Master session ids only; a recovery strand's own session is never part of the run's write evidence. Picked: record recovery session ids per batch and read them into a separate bucket (D4).
4. Fault 3b (exit code). `batch_failed` already exits non-zero; a terminal `stuck` or `dead` digest exits 0. Picked: non-zero for every non-`done` terminal digest, as a refusal with its spec row (D5).
5. Fault 4 (recovery gates). The implementer body's sandbox rule bans mutating `lyx fabric` verbs; the recovery prefix never says which gates stay allowed. Picked: name them in the recovery prefix, plus the `tmux`/`llm` step rule (D6).
6. Faults 5 and 6. quarry resolves same-package test files and unexported names; it does not evaluate build constraints and answers `ambiguous` for a build-split member. #511's targets match #500's shape, so one fix covers both. Picked: a lyx-side build-split classifier read by every planglyph status check (D7), and a done-check detail that names the lookup (D8).

## Rejected alternatives

- D1: lyx stops the shell and records it discarded. Rejected: the provider reports no pid, and shuttle's doc records that lyx kills no shell.
- D1: hold every outstanding shell, not only gate shells. Rejected: a stray `tail -f` or Monitor would hold every run to its timeout.
- D1: forbid Master from running gate tests in its stencil. Rejected as the sole fix: the stencil is not enforced, and the result of a run already started is still lost.
- D3: a lyx verb that spawns the verify fixer. Rejected: Master's forks are in-session Agent-tool forks; a verb could only start a cold strand, a second recovery mechanism for the same job.
- D3: stencil-only change. Rejected: the stencil already states the rule; the contradicting Go text is the defect.
- D4: second recovery starts from a clean tree with the first attempt's diff offered as reference. Rejected: it needs a destructive reset of uncommitted work and throws away the first attempt's state, where keeping it costs nothing the suspect rule does not already guard.
- D4: fold recovery sessions into `recordedSessions`. Rejected: the run-exit audit reads Master sessions' parent writes, so recovery writes would become parent-write findings.
- D7: change quarry to honour build constraints. Rejected: quarry's resolver deliberately refuses to guess a GOOS, and the change would need a quarry release; the policy is lyx's.
- D7: accept every ambiguous answer at done-check time. Rejected: a real duplicate declaration would pass.
- D7: let a Create reuse a name every declaration of which the same card deletes (#507). Rejected under YAGNI: with build-split accepted the symbol is edited in place.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): hold gate shells or stop them? Hold, by awaited prefix `lyx gate `; lyx cannot stop a shell.
- Q2 (resolved, auto-pick): which prefix? `lyx gate ` covers every gate verb; a gate run spelled otherwise is not held and is recorded as unobserved.
- Q3 (resolved, auto-pick): where does the "do not fix it yourself" rule reach Master? In the re-prompt text itself, through a per-entry text function on `GateEntry`.
- Q4 (resolved, auto-pick): attribution by batch or by run? Recovery writes join the run's writes, as fork writes already do; the prompt groups by run.
- Q5 (resolved, auto-pick): does a recovery's write ever count as a parent write? No; it sits in its own bucket.
- Q6 (resolved, auto-pick): which terminal digests exit non-zero? `stuck`, `dead` and `failed`; `done` and running stay 0.
- Q7 (resolved, auto-pick): recovery prompt only, or the shared implementer body too? Recovery prefix only, per the board entry.
- Q8 (resolved, auto-pick): does quarry resolve unexported or test-file names? Yes, per its `unitFor` and match rules; the #511 failure is read as the build-split case.
- Q9 (resolved, auto-pick): what "flag at Plan time" means. The Plan-time status policy and the done-checks share one build-split classifier, so a glyph Plan accepts cannot fail record-batch for this cause.
- Q10 (resolved, auto-pick): revive a dead batch through `lyx loom goto` (#511 point 3)? Out of scope; not on this board entry.
- Q11 (open, plan writer): source of the GOOS, GOARCH and `unix` sets for D7's configuration space (R2).

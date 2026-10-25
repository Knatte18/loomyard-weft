# Support log — regression-baseline-verify

## Interview

The session ran autonomously with no advisors; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Read the board entry (no `Decision for regression-baseline-verify:` note on the board), issue #609, the loom and darn recipes, the entry-gate and Command-row code, the gate-slot, runproc and config-registry docs, and the commits that rebuilt the run tail.
Found that `Parent-Align` has run before the Code perch since `fc75aef79`, which the entry's "before Describe and Parent-Align" predates.

### Turn 2 — design, decided alone

Settled D1–D7 against the entry's design and the operator decisions of 2026-10-25.
The questions that needed a choice are in the ledger as auto-picks Q1–Q9.

### Turn 3 — lens review

Seven forks ran, one per lens, and every one returned a report.
Findings, merged across lenses:

- interactions 1, walkthrough 1: goto Drift-Baseline after a dead or moved capture looped to the same halt — fixed: a running record with a free lock counts as failed, and a halt after HEAD moved names only goto Webster (D3, D5).
- interactions 6: darn's foreground capture had no crash re-entry — fixed by the same rule (D3).
- interactions 2, existing-paths 2, bound-breaker 6: a main capture could run after code changed — fixed: it refuses outside `Drift-Baseline`, off the branch point and on a dirty tree, and `awaited_at` moved to the durable scope record (D4, D5).
- interactions 3: `lyx drift scope` was unbounded — fixed: refused outside `Drift-Baseline` (D3).
- interactions 4, walkthrough 3, bound-breaker 7: the await bound could expire during a slot wait and had no outcome — fixed: unbounded while waiting for a slot, timeout plus one minute from command start, then Stuck (D4, D5).
- interactions 5: side captures' rules were unstated — fixed (D6).
- interactions 7: whether a recapture resets `awaited_at` — fixed: a fresh capture writes a fresh record, and `awaited_at` lives in the scope record, set only on a passed first await (D4, D5).
- interactions 8: only Drift-Baseline checked declared commands — fixed: every row and verb that runs one (D1).
- walkthrough 4, existing-paths 1: nothing committed `_lyx/drift/` — fixed: each row names what it commits (D3, D4, D5, D6).
- walkthrough 2, bound-breaker 4: the compare did not rerun after a gate-driven fix — fixed: the compare runs last on every attempt (D6).
- bound-breaker 5: compare on uncommitted edits — fixed: a dirty tree re-prompts (D6).
- walkthrough 5: `.lyx/` dirt in a hub that does not ignore it — fixed: excluded through `.git/info/exclude` (D4).
- bound-breaker 1: a side capture could overwrite a harness-default baseline — fixed: commands must hold `{baseline_dir}` (D1).
- bound-breaker 2: zero-test pass — fixed for capture (files written); the compare side is a Models fact, moved to Open risks; the all-accepted case is bounded by fix 11 under `full` and is the agent's judgement under `partial`.
- bound-breaker 3: no runproc route for one child, and Windows — fixed: a new child entry, PATTERN change listed, and the Windows hold (D4).
- bound-breaker 8, 9: liveness race and pid reuse — fixed: a lifetime OS lock replaces the pid check, and records are re-read after it (D4, D5).
- bound-breaker 10: "never beside another heavy run" — fixed: "within the hub's slot count" (D4).
- bound-breaker 11: unbounded accept — fixed: under `full` any acceptance is Stuck (D6).
- bound-breaker 12: agent could reset the budget counter — partly fixed: the counter is Go-written under `.lyx/drift/`; rejected beyond that, since agents run under the edit directive and no record in lyx is tamper-proof against them.
- bound-breaker 13, leanness 3: unsourced VSTest exit-code claim — fixed: dropped from Constraints, the pass rule no longer rests on it, and the Models fact moved to Open risks.
- bound-breaker 14, existing-paths 6: darn entry-gate wiring — fixed: a plan-writer note on `Env.SettingKnown`; loom's operand check holds per bound-breaker.
- existing-paths 3: pause and sleep during a run — fixed: the timeout counts on the monotonic clock (D4).
- existing-paths 4: amendments never re-scope — fixed: stated, and the agent judges against the amended record (D6).
- existing-paths 5: the halted row as current row — fixed: stated (D7).
- existing-paths 7: darn's Code perch after Drift-Verify — fixed: stated (D2).
- goal-scope 1, board-fit 1 (`decision`): Drift-Verify's position against Parent-Align — already under Open risks for the parent; no change.
- goal-scope 2: trust check by whole tree rather than `*.cs` — rejected: `_lyx` is a symlink outside the code tree, so record writes never move HEAD's tree, and the branch-point check is stricter than the `*.cs` diff.
- goal-scope 3: Drift-Await's ungated departure unnamed — fixed (D2).
- goal-scope 4: escalation passes the arbiter first — fixed (D2).
- goal-scope 5: no refresh verb — fixed: the pre-change recapture with backup is the only refresh (D4).
- goal-scope 6: budget of 8 may starve — rejected: configurable auto-pick.
- board-fit 2: anchor-relative paths — fixed: `dir` as a gate tier's, paths through `lyxdirs` (D1, Constraints).
- board-fit 3: the C# sandbox hub — fixed: plan-writer note.
- leanness 1, 4, 5, 6, 7: duplicated PATTERN checklist, auto-mode values, notes, testing line and teardown bullet — fixed: trimmed.
- leanness 2: rejection narrative in D4 and D2 — fixed: moved to a Constraint and Rejected alternatives.
- leanness 8: emphasis only — no change.

## Rejected alternatives

- `Drift-Verify` between Webster and `Parent-Align`: keeps parent commits out of the compare but leaves the Code perch's fixes uncompared, against the entry's "after the Code perch".
- `Drift-Await` behind `entry_gate: drift.enabled`: turning the flag off mid-capture would let Webster build beside a running capture.
- Re-capturing automatically at `Drift-Await` after a dead capture: the operator's rule halts on a failed capture, and a goto keeps the choice with the operator.
- A filter-string skip interface (`skip_item`, `skip_join`): harness-specific and awkward when the list is empty.
- An agent-written scope file parsed by Go: would need a sole parser; a verb writing a Go record needs none.
- Refreshing the baseline to accept drift: issue #609 forbids touching a baseline to make a compare pass.
- Config in `loom.yaml`: the rows also run in darn.
- A token or time budget: lyx cannot count or refuse either from inside the agent's session.
- Leaving the agent's fixes to Finalize's landing gate alone: a failure there reaches the merge fixer, not the agent that made the change.
- A capture in a separate checkout: issue #609 rules it out after Millhouse #1181, and the board entry rejects it for .NET build time.
- `Drift-Baseline` after Discussion-Write: the decision record is not yet approved there.
- A pid check for capture liveness: racy and fooled by pid reuse on Windows; a lifetime OS lock is neither.
- Trusting the baseline by `git diff <commit> -- '*.cs'`: the branch-point check is stricter and language-neutral, and `_lyx` lies outside the code tree.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): where `Drift-Verify` sits now that `Parent-Align` precedes the Code perch — after `Code-Bouncer`, before `Describe`; raised to the parent under Open risks.
- Q2 (auto-pick): config home — a new hub-wide `drift` module with `enabled`, `capture`, `compare`, two model keys and a run budget.
- Q3 (auto-pick): skip interface — a `{skip_file}` placeholder naming a file of test names.
- Q4 (auto-pick): baseline location — lyx-owned `{baseline_dir}` under `.lyx/drift/`, backed up rather than overwritten.
- Q5 (auto-pick): how drift is accepted — `lyx drift accept` adds the test to the skip set, only at `Drift-Verify`.
- Q6 (auto-pick): `Drift-Await` gating — ungated, acting on run records.
- Q7 (auto-pick): budget — 8 compare and side-capture runs per agent session.
- Q8 (auto-pick): models — `sonnet[medium]` for the baseline agent, `opus[high]` for the verify agent.
- Q9 (auto-pick): a test gate over `Drift-Verify`'s fixes — the new `drift-verify` gate key, scope `diff`.
- Q10 (open, for Models on OSL-1046): whether its harness reads `NORCE_BASELINE_DIR` in verify mode and a skip file, its capture's exit code, and whether its compare fails on zero tests; recorded under Open risks.
- Q11 (auto-pick, lens review): bound `lyx drift accept` under scope `full` by ending Stuck on any acceptance.
- Q12 (auto-pick, lens review): end a timed-out command through a new `runproc` child entry, a `PATTERN-run-process-owner` change.

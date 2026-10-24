# Support log — gate-runs-once

## Interview

The session ran autonomously with no advisors, so each turn is a batch of questions the writer settled from the code and the board entry.

### Turn 1 — mapping the board entry onto the landed tail

- Which sites run test gates after `loom-tail-review`?
  Read from the code: Webster's card, batch, end (`final`) and verify-fixer (`round`) gates; Merge-In, Finalize, Darn-Write and Code-Burler-Fix through the `landing` list; the merge fixer through `landing`.
  Darn has no Merge-In; its Finalize merges the parent under the lock.
- Is there still a verify row whose entry gate could skip a repeat (item 3)?
  No: every gate runs inside a row with other work. Settled: the carry lives in `lyx gate test` (D4).
- Is item 4 still open? No: the hub file already lists `tier2` under `final` and `landing`.

### Turn 2 — gate keys and scope

- Key naming: row names in lower case for recipe rows, `card`, `batch`, `webster` and `webster-fix` inside Webster (D1).
  `round` renamed under the producer rule, since both its sites are Webster's verify fixers.
- Where the scope resolves: in `lyx gate test --gate <key>`, so sites stop rendering tiers (D3).
- `diff` base: newest ancestor commit with a whole green of the tier, else the parent merge base, else whole (D2).
  Recommended over the round form's last-pass base because a narrowed green never becomes a base.
- Defaults: reproduce today's calls per site; no scope change is made for the operator.

### Turn 3 — the carry

- Granularity: per tier, per tree, per module, per args; whole or a package set (D4).
  Recommended because darn's Finalize `all` after Darn-Write's `diff` then runs only what Darn-Write did not, which is "once per tree".
- Dirty trees: no carry, no record, following `verifytree.DirtyPaths`.
- Override: `-- -count=1` already breaks the carry through the args match, so no new flag.

### Turn 4 — disjoint tiers

- Tag naming: tier names (D5), from the operator's wording.
  The rename adds the already-tagged files to the edit; the measured gain of disjointness is about 20 s of package time per higher-tier run, which the operator decided for.
- Untagged support files (`TestMain`, shared helpers) must stay untagged, since every tier's build needs them; so the rule is about files holding test functions.
- The guard step: 44 of 51 guard-marked files are untagged today and become `tier0`, which `guard_tier: 1` would not compile; settled per-tier guards and retiring `guard_tier` (D6).
- Transition: a hub-wide tier command change is shared by every worktree; dual-tag commands are right for old and new trees, so the change is safe before landing (Constraints); the installed `lyx`'s package loads still miss `tier0` test imports in narrowed gates until deploy, bounded by the whole gates (D13).
- Vacuous passes: a command missing its tag runs nothing and passes; settled the D7 check.

### Turn 5 — row entry gates

- Where evaluated: on entering the row, with an entered marker so a resume never re-evaluates a half-run row (D8).
- Darn's budget of 0: the per-perch budget falls back to `review_max_bounces` when null, which no single setting expresses; settled the `budget.<Segment>` operand (D9).
- Darn's perch: take loom's `zero_rounds` instead of a gate on the Bouncer, and gate only Describe (D10), keeping the Bouncer symmetric between recipes.
- `--window`: a value of battencli's setting lookup rather than an `Env` field, under `PATTERN-recipe-setting-lookup`.

### Turn 6 — measurement

- Source: the gate run records, machine-local, 14-day retention (D11); the rendered logs hold no event times, so `lyx gate test` adds a per-package launch list to the record.
- XDR share: `go test -json`'s `start` event is emitted before the test binary's exec (go doc cmd/test2json, go1.26.8), so the gap to the next event holds the hold; records from before this task carry no launch list.
- Before and after on OSL-1046: the operator's report run there after deploy; noted as an auto-mode assumption.

### Turn 7 — lens review

All seven forks returned.

- interactions BLOCKING, D7 cannot guard this run, whose gates run the installed `lyx`: fixed, D13 and the reworded open risk.
- interactions LOW, carried tier writes no run record: fixed in D4.
- interactions LOW, `code-burler-fix` fallback is not today's `--all`: fixed, D1 names the change.
- interactions LOW, pruning loses an older ancestor base: fixed in D4.
- interactions LOW, old `GuardNotIntegration` refusal not said to go: fixed in D6.
- interactions LOW, guard step's "outside scope" under per-tier bases: fixed in D6, its own tier's scope.
- existing-paths MEDIUM, entered marker against goto, relaunch and rename: fixed in D8, the status names the entered row.
- existing-paths MEDIUM, tackler relaunch bypass unstated: fixed in D8, a relaunch evaluates.
- existing-paths MEDIUM, `RerouteReviewed` may read the edge: rejected, `rerouteReviewed` in `internal/loomcli/landingdeps.go` reads the resolved Code-Perch budget, not the routing.
- existing-paths LOW, caller label moving onto a skip entry: fixed in D8.
- existing-paths LOW, checklist reads `skipped` as a visit: fixed by D8's rule for every history reader.
- existing-paths LOW, carried summary line shape: fixed in D4.
- bound-breaker BLOCKING, installed guard scan refuses retagged guard files, `main`'s gate included: fixed, D13's transitional `|| integration` term and follow-up entry.
- bound-breaker BLOCKING, D7 as backstop false during the run: fixed, merged with the interactions finding.
- bound-breaker MEDIUM, old binary on another machine refuses the reshaped `gate.yaml`: fixed, an open risk with its bound, since every config change shares it.
- bound-breaker MEDIUM, D7 false failure on platform-constrained files: fixed in D7.
- bound-breaker MEDIUM, launch latency includes `TestMain`: fixed in D11, Neve as the reference.
- bound-breaker LOW, tree changing during a run: fixed in D4.
- bound-breaker LOW, evaluation error status visibility: fixed in D8 and Constraints.
- bound-breaker LOW, old binaries reading `skipped`: rejected, `PATTERN-tolerant-records` already binds history readers and an old reader sees an unknown outcome in a finished row only.
- walkthrough BLOCKING, transition fails under the installed `lyx`: fixed, merged into D13.
- walkthrough MEDIUM, D7 misses packages with an untagged `TestMain`: fixed in D7, counted from run events.
- walkthrough MEDIUM, `--card` rendered from a scope read at render time: fixed in D3, `card` and `batch` always pass card ids.
- walkthrough LOW, carried run records: fixed, merged with the interactions finding.
- goal-scope MEDIUM, no before baseline captured: rejected, gate records are machine-local and the XDR machine is not this one; the report reads records from before the landing within retention, recorded as an assumption.
- goal-scope MEDIUM, darn Bouncer departs from the operator view: fixed, D10 names the departure and its reason.
- goal-scope MEDIUM, tag rename widens scope: fixed, D5 states the cost; the pick stays on the operator's wording.
- goal-scope MEDIUM, hub change has no owner: fixed, Constraints names the delivery to the orch as the hand-off, and D13 adds the discovered-count check.
- goal-scope LOW, cost against gain of disjoint tiers: fixed in D5.
- goal-scope LOW, D7 unasked: rejected, it guards D5's tags permanently after deploy.
- goal-scope LOW, D2 merges the two `diff` readings: fixed in D2's rationale.
- goal-scope LOW, `run_settings` question: fixed in D9.
- goal-scope LOW, gopls left out: rejected, the IDE seed is operator config outside the code; kept as an open risk.
- board-fit MEDIUM, row-name keys do not serve pieces that keep darn's row names: fixed, D1's `gate_key`.
- board-fit MEDIUM, a branch landing on `main` during this run: fixed in Open risks.
- board-fit LOW, `regression-baseline-verify` wants rows hidden: rejected, the board entry's decision asks that a skip be visible.
- board-fit LOW, `--window` against `open-window-toggle` and `batten-window-default`: rejected, those entries own that change and D10 keeps today's meaning.
- board-fit LOW, `adopt-runs-tests` key: rejected, already an open risk.
- board-fit LOW, `test-time-audit-recurring` commands change: rejected, D5 names `cmd/testtiming` and the comparability risk is listed.
- board-fit LOW, green record scoper-generic for .NET: rejected, D4 already carries whole entries under any scoper.
- board-fit LOW, XDR entries as consumers: rejected, no design change.
- leanness MEDIUM, lint base hidden in Notes: fixed, moved into D3.
- leanness MEDIUM, D11 evidence only in the log: fixed in D11.
- leanness LOW, assumptions repeat rationale: fixed, assumptions point at D4 and D5.
- leanness LOW, no-change constraints: fixed, cut.
- leanness LOW, D5 restates helper rules: fixed.
- leanness LOW, D12 closing paragraph: fixed, cut.
- leanness LOW, D3 early-refusal line: fixed, cut.
- leanness LOW, open risk restates the constraint: fixed, shortened.
- leanness LOW, history reader list: fixed, D8 states the rule and the list is cut.

## Rejected alternatives

- A verify row's entry gate for the carry (board item 3 as written): no verify row exists after `loom-tail-review`; the gate shares its row with merges and fixers.
- Keeping `GateCall` rendering `--tier` lists and scope words: every site would resolve `diff` itself, and the record would not know its key.
- A common `diff` base for all listed tiers: per-tier bases are as cheap when tiers share a commit and more precise when they do not.
- Whole-only carry: leaves darn's Finalize rerunning Darn-Write's packages over the same tree.
- A negated tag (`!tier1 && !tier2 && !tier3`) for `tier0`: the operator pictured `tier0` tagged by its name, and the negation must grow with every tier.
- Keeping `integration`, `tmux` and `llm` and adding only `tier0`: needs a tier-to-tag map in config for the guard step and D7.
- A single `guard_tier` with guard tests moved into its files: the `tier1` guards need integration fixtures, and moving 44 tests into `tier1` files costs them speed.
- Parsing each tier command's `-tags` to check it names the tier: hubs without tier tags would be refused.
- An entry gate on darn's Code-Bouncer: darn's Bouncer would behave differently from loom's, whose `zero_rounds` also carries the plan and discussion approval seams.
- A `ForcedEntries` field on `Env` for `--window`: `PATTERN-recipe-setting-lookup` bars a row value as its own field.
- Evaluating the entry gate at every call of the row: a resume could skip a row halfway through.
- A new `--rerun` flag on `lyx gate test`: `-- -count=1` already reruns.
- Measuring through `lyx report timing`: it reads git records, while the gate records are machine-local.
- Keeping guard-marked files untagged until the deploy: they hold tests, so they would run in every tier and need a scan exemption no cleaner than D13's term.
- Splitting this task into two landings: the board lock forbids changing this entry's scope; a follow-up entry for the residue does the same with less churn.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where does item 3's carry live? Auto-pick: in `lyx gate test`'s per-worktree green record (D4). Resolved.
- Q2 What are the gate keys? Auto-pick: row names in lower case plus Webster's `card`, `batch`, `webster`, `webster-fix` (D1). Resolved.
- Q3 Does `round` keep its name? Auto-pick: no, `webster-fix`, under the producer rule. Resolved.
- Q4 How does a site name its gate? Auto-pick: `lyx gate test --gate <key>` (D3). Resolved.
- Q5 What is the `diff` base? Auto-pick: per tier, newest ancestor whole-green commit, else parent merge base, else whole (D2). Resolved.
- Q6 Which scope does each migrated key take? Auto-pick: today's call per site. Resolved.
- Q7 Does the carry subtract packages or only skip whole tiers? Auto-pick: subtract packages under the `go` scoper. Resolved.
- Q8 Are tags renamed to the tier names? Auto-pick: yes (D5). Resolved.
- Q9 What happens to `guard_tier`? Auto-pick: retired; guards run per tier (D6). Resolved.
- Q10 How does the hub's tier config change without breaking other worktrees? Auto-pick: dual-tag commands applied by the orch before Webster retags (Constraints). Resolved.
- Q11 Is a vacuous-pass check needed? Auto-pick: yes, D7. Resolved.
- Q12 When is an entry gate evaluated? Auto-pick: on entering the row, with an entered marker (D8). Resolved.
- Q13 Does `lyx shed goto` honour the gate? Auto-pick: no, the shed goto verb is a deliberate force, whoever calls it. Resolved.
- Q14 How does darn's gate read a per-perch budget with a null fallback? Auto-pick: the `budget.<Segment>` operand (D9). Resolved.
- Q15 Which darn rows get gated? Auto-pick: Describe only; the Bouncer takes `zero_rounds` (D10). Resolved.
- Q16 How is `--window` forced? Auto-pick: a value of battencli's setting lookup (D10). Resolved.
- Q17 Where does the measurement live? Auto-pick: `lyx gate report` over the machine's run records and logs (D11). Resolved.
- Q18 Who runs the OSL-1046 measurement? Auto-pick: the operator after deploy. Resolved.
- Q19 How do retagged guard files pass the installed `lyx`'s guard scan? Auto-pick: `tierN || integration`, removed by a follow-up entry after deploy (D13). Resolved.
- Q20 How does a recipe keeping darn's row names get its own gate keys? Auto-pick: a row config `gate_key` (D1). Resolved.
- Q21 Does a tackler relaunch bypass the entry gate? Auto-pick: no, only the shed goto verb does, whoever calls it (D8). Resolved.
- Q22 How does darn's tree-scoped verify name its gate key? Auto-pick: `tree_scope_verify` retires and the sites pass `--gate` with the touched trees' targets (D14). Resolved.

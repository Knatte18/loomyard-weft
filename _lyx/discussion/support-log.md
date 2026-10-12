# Support log — judge-cap-call

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry and the related entries `go-driver-assistant`, `arbiter-when`, `arbiter-relaunch-moved`, `orch-split` and `report-timing`.
   The entry settles the hard cap's default (4), its own key beside `review_max_bounces`, per-perch overrides, and that hard-cap and returning-blocker stops go to the arbiter through the existing escalation.
2. Explored the Bouncer (`internal/shedadapters`): the budget path escalates with cause `budget` or auto-accepts; continues with cause `budget` are budget-exempt; `BlockingEscalates` exists only for Merge.
   Explored loom and darn config, the judge, escalation and arbiter stencils, and the arbiter's eligibility.
3. Settled the tag (D1): asked every round, missing reads as `design-wrong`.
4. Settled the rule's order (D2), the hard cap's keys and refusals (D3), budget exemption past the soft cap (D4), the verb refusal (D5), the new cause (D6), the arbiter's rule (D7) and auto-accept's fit (D8).
5. Lens review: six forks spawned before writing; all six returned.
   Duplicates across lenses are merged into one line.
   - goal-scope, scenarios, premises (MEDIUM): an earned `CIRCLING` at the soft-cap round fell to continue — fixed, D2 step 4 escalates it from the soft cap on.
   - goal-scope (MEDIUM): `design-wrong` at any severity widened "blocker" — fixed, only BLOCKING and MEDIUM count (D2, A3).
   - pattern-fit, premises (MEDIUM): `PATTERN-tolerant-records` mis-cited; escalation and decision-file cause readers refuse a third cause; older binaries — fixed, cite dropped, D6 names both readers and writers, open risk added.
   - scenarios (MEDIUM): a recorded continue at or past the hard cap ran round H+1 — fixed, D2 escalates `hard-cap`, acceptance criterion added.
   - scenarios (MEDIUM): a parse-error facts row could be accepted unreviewed — fixed, D2 step 1 escalates `budget`.
   - scenarios (MEDIUM): unreadable previous ledger undefined — fixed, D2 step 1.
   - scenarios, premises (MEDIUM/LOW): auto-accept replay path unstated — fixed, D8 sends replays to the rule.
   - board-fit, goal-scope (MEDIUM/LOW): D7 overlaps `arbiter-when` unnamed — fixed, Scope names the item taken over.
   - board-fit (MEDIUM): D7's `budget` rule against `arbiter-when`'s "never budget" — fixed, D7 leaves eligibility to `arbiter-when`.
   - board-fit (MEDIUM): `go-driver-assistant` dependency unnamed — fixed, Scope "Out" states the causes reach the arbiter and Go driver through the unchanged list.
   - board-fit, goal-scope (MEDIUM/LOW): orch role stencil's `circling continue` and small-task advice — fixed, added to Scope.
   - premises, pattern-fit (MEDIUM/LOW): darn is hub-wide; its soft cap 0 skips review — fixed in D3.
   - pattern-fit (LOW): run-settings category lives in `loomengine`'s spec — fixed in D3.
   - pattern-fit, scenarios (LOW): refusal way forward should name the config route; parked runs — fixed in D3, D5 and Open risks.
   - pattern-fit (LOW): arbiter stencil should point at the judge's tag meanings — fixed in D7 and Constraints.
   - pattern-fit (LOW): refusal-spec row placement — fixed in D3.
   - premises (LOW): `blocking_escalates` is a recipe row key — fixed in D8.
   - premises (LOW): Awaiting Reason offered continue at `hard-cap` — fixed in D6.
   - scenarios (LOW): D7 `budget` silent on continue — fixed, `budget` settles as `circling`.
   - scenarios (LOW): Bouncer and verb may read different hard caps — fixed, D3 says both resolve from the same config load at call time.
   - scenarios (LOW): missing acceptance scenarios — fixed, added.
   - goal-scope (LOW): Goal overstated — fixed.
   - goal-scope (LOW): darn addition — kept as A5, Scope now names the Darn-Review perch.
   - leanness (MEDIUM): D2 bound and Open risk 1 repeated — fixed, risk trimmed.
   - leanness (LOW): assumptions repeated decisions — fixed, one pointer line each.
   - leanness (LOW): sole-parsers bullet, duplicate no-conditional and tolerant-records lines, test placement in Constraints — fixed.
   - leanness (LOW): question ledger mirrors assumptions — rejected, both sections are required in different files.
   - board-fit (LOW): `report-timing` needs a rule-vs-auto-accept `by` value — rejected, reports are out of scope and the decision file already marks automatic decisions.
   - board-fit (LOW): name `twill-plan-followups` in Open risks — rejected, it measures lower budgets, which fit under the hard cap.
   - board-fit (LOW): `arbiter-relaunch-moved` relationship — rejected, the two do not touch.

## Rejected alternatives

- The judge decides accept or continue itself, writing a new verdict value: rejected, since Go then could not enforce the hard cap and the returning-blocker check against a verdict, and the judge's output format would grow a second decision channel.
- Tag only from the soft-cap round on, through a Go-held stencil variant: rejected for a uniform ledger and a simpler stencil; the tag costs one field per open entry.
- A missing tag reads as `mechanical`: rejected, since it would accept unreviewed design fixes on a judge's omission.
- Raise a hard cap below the soft cap to the soft cap silently: rejected; a refusal makes the operator see the conflict.
- Darn takes loom's hard cap, or its hard cap equals its soft cap: rejected; darn has its own budget key, and the hard cap belongs beside it.
- Narrow the arbiter's eligibility to `hard-cap` and `circling` here: rejected as `arbiter-when`'s scope.
- Drop the `budget` cause: rejected; an unreadable ledger still needs a stop the rule cannot settle.
- Make auto-accept follow the rule on a harvest too: rejected; it is a per-perch operator choice the entry does not reopen.
- Let any severity of `design-wrong` count: rejected; the entry speaks of blockers, and a LOW finding should not buy rounds.
- Apply the returning-blocker and `CIRCLING` checks only past the soft cap: rejected; it would turn an earned circling call at the soft-cap round into a continue.
- Raise a `run_settings` hard cap through `lyx board upsert` and `lyx batten run <slug>` alone: rejected; Seed-Child runs once, before either refusal can arise, and a later `batten run` resumes at Run-Shed, so the board edit never reaches the task's local layer.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): when does the judge tag findings? Every round (D1, A1).
- Q2 (auto-pick, resolved): what does a missing tag mean? `design-wrong` (D1, A2).
- Q3 (auto-pick, resolved): which severities count as design-wrong? BLOCKING and MEDIUM (D2, A3).
- Q4 (auto-pick, resolved): what is a returning blocker? A key that counts as design-wrong in this round and the previous one, after a continue granted on the previous round; an earned `CIRCLING` escalates from the soft cap on (D2, A4).
- Q5 (auto-pick, resolved): does darn get a hard cap? Yes, its own darn config key (D3, A5).
- Q6 (auto-pick, resolved): hard cap below soft cap? Refused at load (D3, A6).
- Q7 (auto-pick, resolved): how do the auto-accept flags fit? Unchanged on a harvest; replays and `blocking_escalates` rounds go to the rule (D8, A7).
- Q8 (auto-pick, resolved): may anyone continue past the hard cap? No; the verb refuses and a recorded continue there escalates (D2, D5).
- Q9 (auto-pick, resolved): does the arbiter's eligibility change? No (A8).
- Q10 (auto-pick, resolved): does the orch's small-task advice set a hard cap? Yes, 3, beside `review_max_bounces: 2` (D9, A9).

# Support log — lighter-test-rules

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — scope and the keep-or-drop line

Read the board entry, its operator decisions (drop wording claims entirely; keep markers, Fill round-trips, PATTERN scans, registry), and the touching entries `stencil-intent-audit`, `creel-agents`, `webster-deviations-list`.
Read `contracts/stencils/*_test.go`, the template and prompt test list, `loom-plan-spec.md` rule 3, `PATTERN-test-economy.md`, the plan-review rubric's test-economy bullet and `test-budget.yaml`.
Found claims_test rows that interpolate Go constants (`MessageTail`, `ParkMarkerFileName`, `SlotBusyExit`) and webster template pins against Go field sets: structure, not prose, so D1 draws the line at "comes from Go or is read by Go".

### Turn 2 — where the rule lives and what replaces it

No stencil or `CLAUDE.md` text asks for a stencil test; the plan spec's coverage rule and the claims file's protected-claims header do.
Picked D3's edits and D4's webster-review rubric bullet as the review guard; picked a PATTERN background bullet over a new entry to spare headroom.
Picked leaving test-budget rows alone (D5) because concurrent tasks may add tests to the same packages.

### Turn 3 — lens review

Six forks reviewed the first draft; all returned.

- premises: Scope's named test files are mostly not prose assertions (`gate_test.go` is Go-built text) — fixed, Scope sorts by D1 instead of by file.
- premises, goal-scope, pattern-fit, scenarios: D2 left the discussion-spec heading rows to Plan — fixed, they go (parser list unexported, partial, pinned by its own tests).
- premises, scenarios: D4 measured against the decision record, which the webster-review rubric does not use — fixed, the plan card's Intent (webster) and board entry (darn).
- premises, pattern-fit: webster-review rubric already has a Test economy list whose "no coverage" item would still demand prose tests — fixed, D3 amends that list and D4 adds only the read-against-the-stick bullet.
- pattern-fit (BLOCKING): `loom-template-plan.md` restates the coverage rule unqualified — fixed, D3 names it.
- pattern-fit: D1 and D2 disagreed on literal Go-read key names — fixed, D1 keeps them only when read from the Go definition.
- pattern-fit: marker order unclassed — fixed, kept, rows join `markers_test.go`.
- scenarios: `mustNot` claims unclassed; twill host-neutral is a cross-stencil rule — fixed, host-neutral kept as a scan, one-stencil bans go.
- scenarios: darn landings bypass the webster-review rubric — fixed, D3 and D4 cover the darn-review rubric too.
- scenarios, goal-scope, leanness: acceptance criterion 1 reached into spec prose and contradicted D1's keeps — fixed, it now says "no assertion D1 drops", spec scope stated in Scope Out.
- scenarios: D2 table empty or naming an unregistered stencil — fixed, criterion added.
- scenarios, board-fit: concurrency risk one-directional and missing merged prose assertions — fixed, Open risks covers both.
- scenarios, leanness: branch-anchoring rule sat in Open risks — fixed, moved into D1.
- goal-scope: Scope Out "rewording any stencil" contradicted D3/D4 — fixed.
- goal-scope: D4 vs board's "Burler review" — fixed, D4 says Burler reads these rubrics.
- leanness: Auto-mode assumptions re-argued option 1 and restated Scope/D2; Constraints carried two non-constraints; Goal restated D1; D3's CLAUDE.md line was not a decision — fixed, all trimmed or moved to Scope Out.
- board-fit: `parentdirective_test.go` scans are hit by `creel-agents` rewording — fixed, note for the plan writer.
- board-fit: D5 premise confirmed by `creel-core` raising budget rows — no change.
- premises, pattern-fit: PATTERN headroom and scan locations confirmed — no change.

## Rejected alternatives

- Keep a short list of load-bearing phrases per stencil (board option 1): superseded by the operator's later decision to drop wording claims entirely.
- Drop the per-card test requirement only for stencil-only changes while keeping existing claims (board option 3): keeps the cost of every existing claim on each reword.
- Delete every claims_test row, the Go-value rows included: a renamed Go constant would then break an agent's file or message contract unseen.
- A new `PATTERN.md` entry for the rule: costs headroom for a rule that is a case of `PATTERN-test-economy`.
- A mechanical scan refusing new prose assertions: YAGNI; review enforces the rest of test economy.
- Lowering test-budget rows: fails concurrent tasks at Merge-In for no gain.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): what sorts one assertion? Comes from Go, read by Go, or a cross-stencil rule stays; one stencil's static prose goes (D1).
- Q2 (resolved, auto-pick): where do kept claims_test rows go? Go values to one table, marker order to `markers_test.go`, twill host-neutral to one scan; discussion-spec headings go (D2).
- Q3 (resolved, auto-pick): where does the rule change? Plan spec rule 3, plan stencil, PATTERN-test-economy background, plan-, webster- and darn-review rubrics, test-file headers (D3).
- Q4 (resolved, auto-pick): what guards wording now? A webster- and darn-review rubric bullet against each rubric's own measuring stick (D4).
- Q5 (resolved, auto-pick): lower test-budget rows? No (D5).
- Q6 (resolved, auto-pick): are Go-built prompt or refusal strings in scope? No; they are Go behavior.

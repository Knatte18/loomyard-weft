# Support log: twill-fork0

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

### Turn 1: exploration

Read the board entry and the entries that touch it (`overview-seats`, `fan-master-preread`, `fan-prompt-tuning`, `loom-model-key-names`), the fork0, fixer and rationale bench reports, the prototype fragments in the prime's `.scratch/twill-fork0-bench/setup/`, the twill directive stencil and renderer, `internal/loomengine/twill.go` and `plan.go`'s twill wiring, the plan gate's lens-review check, the `twill-plan` fan and its lenses, `burler-step-fix.md`, and the twill-plan bench's J6 key item for the wrong-parent-directory miss.

Findings that shaped the design:

- The directive is host-neutral by test; fork0 is Plan-only, so its text must be an optional part filled by host values.
- No fork audit runs for Plan-Write, so fork0 needs no audit change; `ForkSubagents` is already set under `plan_producer: twill`.
- The overview-last rule already marks a complete plan, so fork0's relaunch needs no new state.
- The hub's `burler.yaml` overrides no twill fan, so the new lens reaches the hub through the template.

### Turn 2: design settled

Decisions D1 to D9 as in the decision record.

### Turn 3: lens review

All six forks returned a report. Premises re-checked against `overview-seats-weft/_lyx/discussion/decision-record.md`, `internal/loomshed/planwrite.go` (`NewPlanDirRotator`) and the completion signal before judging.

- board-fit, BLOCKING, record written on `plan_producer`/`discussion_producer`, which `overview-seats` retires: fixed as proposed; D0 added, D1, Constraints and Acceptance criteria restated on `plan_lenses`/`discussion_lenses` and the lead seat.
- board-fit, MEDIUM, merge conflict with `overview-seats` on the twill hosts and tests: fixed as proposed, merged into D0 and an Open risk.
- board-fit, MEDIUM, judge with advisors at Discussion-Write unstated: fixed as proposed; D6 says the judge is the lead's step, the criterion covers lead with and without advisors.
- board-fit, LOW, `branch-coverage` overlaps `criteria-coverage`: fixed differently; dropping `criteria-coverage`'s branch clause would remove its fidelity check on record-named failure exits, so D7 states the overlap as intended instead.
- board-fit, LOW, `lighter-test-rules` unnamed: fixed as proposed in the test-economy constraint.
- pattern-fit and scenarios, MEDIUM (merged), a master ending after the overview exists counts as finished under `PATTERN-completion-signal` and is re-prompted, not relaunched, and the re-prompt rule would skip the lens review: fixed as proposed; D3 rewritten around the completion signal, with a re-prompted master that has not spawned its lens forks running the lens review once.
- scenarios, MEDIUM, relaunch is a fresh spawn whose rotator archives the partial plan: fixed as proposed; D3 says a session ending before the overview exists is respawned fresh and runs the whole procedure.
- scenarios, LOW, the "at most twice" bound did not hold under a second relaunch: fixed differently; the bound is now per master session, a respawn counting as a new master, as today.
- scenarios, LOW, D3 branches as one acceptance item: fixed as proposed; each failure path is its own criterion item.
- scenarios, LOW, the plan rejection rule's "unplanned test" escape dropped: fixed as proposed in D6's waiver sentence.
- premises, MEDIUM, `hostNeutralClaims` bars "decision record", "Card Index", "validate-plan" and more, which D2/D3's fork0 text uses: fixed as proposed; D4 names every host-specific noun as a host value.
- premises, LOW, the discussion template's marker placement unnamed: fixed as proposed in the Notes.
- goal-scope, MEDIUM, D6 weighed the premise "against what was read" instead of the code and the decision record: fixed as proposed, with the governing input a host value.
- goal-scope, LOW, D7's unreachable-row clause duplicates `row-reachability`: fixed as proposed, clause dropped.
- goal-scope and pattern-fit, LOW (merged), D4 grew prose in the Go spawn-point constant against the stencil-ownership constraint: fixed as proposed; the fork0 sentence moves into the stencil part.
- goal-scope, LOW, D5 had no criterion: fixed as proposed.
- goal-scope and leanness, LOW (merged), unsourced lens cost and bench tallies: fixed as proposed; figures dropped, reports cited.
- pattern-fit, LOW, retiring `rejection_rule` changes the marker list pinned by `markers_test.go`: fixed as proposed in D6.
- leanness, LOW, no-op Constraints bullets: fixed as proposed; removed, the model and skill points folded into D2.
- leanness, LOW, the bench-strength Open risk: fixed as proposed, removed; the bench report records it.
- leanness, LOW, D6 copies the prototype text: disputed, every fix worse; the fragments sit in the prime's git-ignored `.scratch/` (board entry, Evidence), so a delta-only D6 rests the requirement on a file the plan writer may not have.
- leanness, LOW, Auto-mode assumptions repeat the ledger: disputed, every fix worse; the spec requires the section in the record, and the plan writer never reads the support log, so each reader needs its own copy.

`overview-seats` dependency: reported to `ly:orch`, since the board entry's `depends_on` is outside this role's write surface.

## Rejected alternatives

- **A third `plan_producer` value (`twill-fork0`) beside `twill`:** keeps the after-flow selectable, which nobody asked for; the operator chose fork0.
- **fork0 text as Go constants in `internal/loomengine/twill.go`, as `planTwillSpawnPoint` is:** puts a long prompt in Go against `PATTERN-stencil-ownership`.
- **A Plan-only stencil for fork0 under `loom/`:** works, but splits the twill flow across two stencil families; an optional part of the twill directive keeps it with the procedure it changes and open to another host.
- **The critical judge at Plan-Write only, Discussion keeping "fix or reject with a one-line reason":** two judge texts in one shared directive, and the weaker one where Discussion-Burler-Fix already uses the two-sided weighing.
- **Respawning fork0 when it returns an incomplete plan:** fork0's context is gone; the master holds the exploration and the partial plan on disk and can finish it, at lower cost and bounded spawns.
- **Burler's deferral disposition in the twill judge:** the judge's write surface is the reviewed files, so no sound change is out of reach.
- **The wrong-parent-directory sentence in the plan rubric:** reaches Plan-Review too, but the board item names the twill lens miss, and the rubric is read in every review round.
- **Narrowing `criteria-coverage` to drop its record-named branch clause, so it and `branch-coverage` split cleanly:** loses `criteria-coverage`'s fidelity check on record-named failure exits; the overlap is stated instead.
- **A mechanical check that each lens-review line carries a disposition:** a new gate under `PATTERN-gate-self-check-parity` for a format no parser reads; nothing asked for it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Does the critical judge also apply where twill reviews Discussion-Write? Picked yes (D6): one judge text in the shared directive, and Discussion-Burler-Fix already weighs findings this way; the benches measured Plan-Write only, recorded under Open risks.
- Q2 (auto-pick): Keep the after-flow selectable? Picked no (D1): the operator chose fork0, and YAGNI.
- Q3 (auto-pick): Where does fork0's text live? Picked an optional, host-neutral part of the twill directive (D4).
- Q4 (auto-pick): What happens when fork0 returns an incomplete plan or the master is relaunched? Picked D3: the master finishes the plan; a relaunch respawns fork0 once only when the overview is missing.
- Q5 (auto-pick): Lens or rubric for the wrong-parent-directory miss? Picked the `criteria-coverage` lens (D9).
- Q6 (auto-pick): Where does `branch-coverage` sit in the fan? Picked after `criteria-coverage` (D7); order has no behavioral effect beyond the rendered list.
- Q7 (auto-pick): Fork boilerplate wording under fork0? Picked a host-neutral "written and passed the self-check before you were spawned" (D5).
- Q8 (auto-pick): Plan against today's keys or against `overview-seats`' axes? Picked `overview-seats` (D0): it is in Webster and retires the keys this design would hang on.
- Q9 (resolved by `ly:orch`): The board entry lists no dependency on `overview-seats`. The orch holds the run before Plan-Write with a loom pause (the entry is claimed), merges main in once `overview-seats` lands, and confirmed the record is to be written on its axes.

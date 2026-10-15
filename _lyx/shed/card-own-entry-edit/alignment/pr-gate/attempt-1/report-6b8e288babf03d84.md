---
verdict: APPROVED
---

The diff of pull request 666 at `4beaa651cc63c086ae4ea1cff83a3361f2c32109` against `main` carries the board entry's goal and every recorded decision, and nothing outside them.

## Against the goal and decisions

- D1: `CLAUDE.md`'s "Docs land in the same commit" rule drops "a design whose code has landed moves from the board entry into `doc.go`", says the building task writes `doc.go` and never changes its own entry's body or scope, and states that no decision, card or step of a run does so.
  `docs/overview.md`'s documentation-lifecycle bullet says the same, and its heading is unchanged.
- D2: `loom-template-discussion.md` forbids a decision that changes the task's own entry's body or scope.
- D3: `loom-template-plan.md` forbids such a card and widens the post-Discussion route to a recorded drop of an own-entry step only, stopping with no parent.
- D4: `darn-template-write.md` leaves the step out, asks the parent to record the drop, names it as an unrecorded parent correction until recorded, stops with no parent, and admits own-entry steps only.
- D5: the Discussion-review, Plan-review and darn-review rubrics each name the breach as a finding, and the Plan-review and darn-review lines accept the recorded drop as the record says.
- Every new rule says "body or scope" and exempts the status write, `run_settings`, other entries and board-module code.
- The scope's Out list holds: no change to the board lock, `lyx board upsert`, `lyx loom validate-plan` or `PATTERN.md`.

The operator decision in the board entry asks for the rule to be made plain where the planner reads it, with a plan check optional; the record declines the check with a reason, which the entry permits.

## Carried-over risks

- `d4-darn-omission-self-contradiction` (Discussion-Review round 3): the fixer gave D4 one ordered account, and the darn write stencil's text follows that account.
- `darn-review-evidence-clause-conflates-requested-step` (Plan-Review round 1): the fixer separated the change description's claim from a record step's request, and the darn-review rubric text carries both sentences.
- Runs already in flight keep their seeded stencils until the next deploy; this is expected and needs no change.

## Overlap

- `engine-usage-readers` (row 15): changes `internal/shuttleengine*`, `tools/tokencount`, `pattern/PATTERN-shuttle-provider-seam.md` and `cmd/lyx/testdata/test-budget.yaml`; no file or behaviour shared with this run.
- `webster-deviations-list` (row 9): its record changes `contracts/stencils/webster/webster-body-implementer.md`, `contracts/specs/loom-plan-spec.md` and `planglyph.ScopeGuard`; this run edits none of those, so there is no overlap.
  Its open risk expected this run to touch `loom-plan-spec.md`, which it does not.
- Parent commits: none listed.

No overlap, so no landing order applies.
This run is at row 22, ahead of both peers.

No board entry is made obsolete by this run.

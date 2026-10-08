# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record and held; all five are fixed.
`lyx loom validate-plan` passes on the fixed plan, with informational findings only.

## Fixed

- **F1** (card 9): the read-error refusal is split in two.
  A malformed or unreadable decision file refuses naming the file, with the way forward to fix or delete it and then re-run `lyx loom resume`.
  A failed recipe lookup refuses with the circling verbs' rebuild-or-reinstall way forward, then a re-run of `lyx loom resume`.
  The `refusal-spec.md` bullet adds the rows `resume: circling decision unreadable` and `resume: recipe lookup failed`.
  `resume_test.go` gains a row for each: a malformed decision file, and a failing recipe lookup injected through a Bouncer-subdir lookup function, which the verb takes the same way the circling verbs take their `bouncerSubdir` dependency.
- **F2** (cards 3 and 7): card 3 now edits `internal/battencli/wire_seams_test.go`, adding `CarryOver` to `intentionallyNil` as a loom-only seam.
  Card 7 now edits `internal/testkit/envkit/envkit.go`, adding `RowReviewModels` to `nilLegal` beside `SegmentBounces`, since a nil map is the documented fallback.
  Confirmed against `envkit.walkNil`, which reports nil func and map fields, and batten's `wire`, which fills neither.
- **F3** (card 9): `internal/loomcli/cli.go` and `docs/overview.md` join the Edit list.
  The Intent names the loom root `Long` text on the `circling` subtree and the overview's escalation paragraph, where both the fork's resume and the batten-driven resume move to `lyx loom resume`.
- **F4** (card 1): the result check now follows the added D2 decision.
  It requires the same H2 sequence, at most one well-formed pair per segment (exactly one for the written segment, none after a removal), and every line outside the replaced span equal to the prior file's line.
  The replaced span is defined for replace, append and removal.
  The existing append/replace/remove test rows assert the outside-span equality, and no new test is added.
- **F5** (card 6): the `escalation_test.go` edit and its Edit-list entry are dropped, because the test sets its own budget and never reads the template.
  The card now says why that test keeps its constant: `config_test.go` pins the template at 3, and the escalation test pins budget escalation at the budget's last round, so the D3 acceptance criterion is still covered.

## Disputed

None.

## Deferred

None.

# Relevant

- `internal/loomcli/validate.go` `validateDiscussionCmd`, `validatePlanCmd` — the two verbs whose checks and help text change
- `internal/loomshed/gates.go` `ValidateLoomDiscussion`, `ValidateDarnDiscussion`, `NewDiscussionGate`, `NewDarnDiscussionGate`, `NewPlanGate` — the shared functions the gates and verbs call
- `internal/loomshed/plancontext.go` `PlanContext` — the plan gate's context checks, where D6 to D8 join
- `internal/discussionparser/validate.go` `Validate`, `Finding`, `missingSections` — the record's existing checks and finding type
- `internal/discussionparser/doc.go` — sole parser of the record, stdlib-only by design
- `internal/planglyph/doc.go` — owner of quarry calls and the plan gates' resolve-backed pass
- `internal/planindex/planindex.go` `Index`, `Finding`, `ErrQuarryUnavailable` — the code index seam and plan finding type
- `internal/impactset/targets.go` — parses `//lym:guard`, the model for the seam marker
- `internal/testkit/envkit/envkit.go` `NilSeams` — reflective seam helper the scan names
- `internal/loomcli/landingdeps_test.go` — a reflective seam test of a `Deps` struct
- `contracts/specs/loom-discussion-spec.md` — the record format the new rules join
- `contracts/specs/loom-plan-spec.md` — the card format and validation checks the plan rules join
- `contracts/stencils/loom/loom-template-discussion.md` — the Discussion writer stencil that gains a pointer
- `contracts/stencils/darn/darn-template-write.md` — the darn writer stencil, pointing at the discussion spec
- `pattern/PATTERN-test-speed.md` — tier and `//lym:guard` rules the seam marker joins
- `docs/research/review-setup-audit-2026-10-28/report.md` — the finding themes behind each check

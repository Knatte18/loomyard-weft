# Round 3 fixer report — discussion

## Changes

- **F1** (fixed as proposed): D4's Rationale adds two lets-through items, a loom run with `webster_auto_accept` true and a round at or past the hard cap (`webster_hard_max_bounces`, or `review_hard_max_bounces` when it is 0, and darn's `review_hard_max_bounces`).
  The bound sentence now says the reviewer reads stencil changes against the stick and a finding is fixed or escalated unless the perch's auto-accept or hard cap accepts the round; the darn clause names only the hard cap, since darn has no auto-accept key.
  Premise checked against `internal/loomengine/template.yaml`, `internal/loomengine/config.go` and `internal/darnengine/template.yaml`.
- **F2** (fixed as proposed): D2 gains a bullet before `every other row goes`: a row naming a marker without pinning its position, for a stencil no Fill round-trip or marker-set test reads in its real file, becomes a row of `TestStencils_TopLevelMarkers`, one row per stencil.
  It names `landing-template-describe-darn.md` as such a stencil and has the plan writer sort each marker-naming row by whether another test reads the real stencil's markers, so the marker-removal acceptance criterion holds for every stencil the rows cover today.
  Premise checked: `internal/landingshed/describe_test.go` writes `darnStencilFixture`, and `markers_test.go`'s table lists no darn Describe stencil.
- **F3** (fixed as proposed): D2's section-scoped definition is narrowed to a `section:` row whose `must` is exactly one marker, lists the rows that stay (`{{.twill_directive}}`, `{{.decision_record_path}}`, `{{.support_log_path}}`, `{{.parent_notify}}`, `{{.arbiter_command}}`), and states that the `{{.specs_dir}}/loom/loom-discussion-spec.md` Step 5 row goes.
  Premise checked at `contracts/stencils/claims_test.go:321`.
- **F4** (fixed as proposed): D2 states the `{{.kickstart_pack_inline}}` row as adjacency (`{{.kickstart_pack_inline}}\n\n## The sandbox rule`), and the acceptance criterion now reads `moving a position-pinned marker out of its anchored place (out of its heading's section, or off the end of the body or its heading)`.

## Disputed

None.

## Deferred

None.

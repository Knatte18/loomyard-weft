# Lens review

Every fork returned a report.

## criteria-coverage

- MEDIUM, attach-after-deploy untested: fixed. Cards 12 to 15 build the old spec beside the table and assert equal output files and fields; card 17's harvest smoke test is named as the case of a run started in the old spec's shape and attached by the seat producer.
- MEDIUM, "only Discussion-Write's prompt changes" untested: fixed. Cards 12 to 15 add byte-equality rows between the lead's prompt from `seatengine.Prompts` and the old spec's prompt; card 18 drops them along with the old builders.
- MEDIUM, reroute into the review row not placed: fixed. Card 5 adds the placement rule and a row.
- MEDIUM, `verdict unreadable` unreached: fixed. Card 6's reader subtest adds an attempt with an undecodable envelope.
- MEDIUM, gate stamp unasserted for Rework, Describe and Darn: fixed. Cards 13, 14 and 15 assert `GotTables[0].Gate`.
- LOW, goto before round 1 and the open visit after a bounce: fixed. Card 5 adds both rows.
- LOW, `claimed-scope-edit` dropped silently: fixed. The overview states no such row exists in this tree.
- LOW, `DiscussionLenses` field doc: fixed in card 19.

## test-economy

- LOW, `TestMigrateConfig` belongs in the configsync table: rejected. That table pins one generic migration per module; the D11 combination rows read loomengine's own rewrite rules and its layered load, which no per-module row covers.
- LOW, card 12's resume row repeats producer-level checks: rejected. The row pins that the Plan-Write rotation leaves the stale plan file in place on resume, which the producer test, holding no rotation, cannot show.
- LOW, card 10's start-error row should be a field of the existing row: fixed. The half that suggested editing card 17's test in place is rejected: the test's name cites the deleted producer.
- LOW, card 6 and card 7 file readers placed as renderer table rows: fixed. Both are now separate subtest blocks inside `TestRenderChecklist`.
- LOW, card 9's probe-prompt assertion cannot fail: fixed. The assertion is dropped.

## row-reachability

- BLOCKING, card 17's rerun smoke test: the second stand-in run would hold the lead's strand name. Fixed: that run takes the role `other`.
- MEDIUM, card 9's one-seat hold-deferral row does not fit the advisor table: fixed. It is now its own block with zero advisors.
- LOW, card 6 and card 7 reader rows: fixed, as under test-economy.
- LOW, card 12's resume row needs scripted `ResumeResults`: fixed.
- LOW, card 11's live-lead row must be scoped to the `discussion` prefix: fixed, with `ProbeFn` and `ResumeFn` named.
- LOW, card 8's stencil seeding unnamed: fixed. It names `stencilkit.SeedInto` for the MultiLLM tests and the fixture-recipe test.

## forced-edits

- BLOCKING, `shedbuild#engineMinimalConfig`'s MultiLLM entry breaks at card 8: fixed. Card 8 lists it, card 11 drops its DiscussionSeats entry, and card 17 drops its SingleLLM entry.
- BLOCKING, `shedbuild#TestBuild_Errors` has a SingleLLM row: fixed in card 17.
- MEDIUM, `loom-status-spec.md` still says only a reroute entry carries `routed_to`: fixed in card 1.
- MEDIUM, the `PATTERN.md` agent-name entry line: fixed in card 8.
- MEDIUM, `landingshed/doc.go` and `darnengine/doc.go` name the deleted builders: fixed in card 18.
- LOW, stencil header comments naming the deleted builders: fixed. Card 18 lists the three landing and darn stencils, and card 11's intent names the discussion stencil line.
- LOW, `loomengine/review.go`'s header: fixed in card 18.
- LOW, `composeActivity`'s doc comment: fixed in card 1.
- LOW, `reworkPaths` may be left dead: fixed. Card 13 says `ReworkTable` builds its paths through it.
- LOW, stale test-file comments: fixed. Card 17 lists the three files.

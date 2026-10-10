# Plan-Review round 1 — fixer report

Every finding was checked against the code and the decision record, and every premise held.
Seven are fixed in full; the absent-key half of `d4-deviations-unrecorded` is deferred, and its resolve-at-load half is fixed.
`lyx loom validate-plan` passes on the fixed plan, with only the informational `create-new-unit` for the advisor stencil.

The plan now has nine cards: a new card 6 holds the fake seat runner's gate-once change, and the old cards 6, 7 and 8 are now 7, 8 and 9, with their files renamed and the Card Index, Shared Decisions and cross-card references renumbered.

## Fixed

### chair-include-breaks-single-agent-composer (BLOCKING)

Confirmed: `loomengine.composePrompt` calls `stencil.FillOptional`, which is `FillWith(template, nil, …)`, and `attachIncludes` refuses an include with no entry.
Card 4 no longer adds `{{template "seat-directive-chair"}}`.
Its chair-stencil changes are now marker-free prose, which the single-agent composer keeps rendering, and its Intent says why the include is deferred.
Card 8 (retire-discussion-spec) adds the include and drops `composePrompt`/`DiscussionSpec` from the stencil header, in the same card that deletes the composer.
It also edits `plan:internal/loomengine#TestDiscussionTable` with a row asserting that the chair prompt names every advisor strand and notes path.
Card 5 drops that assertion and points to card 8.
The Shared Decision `chair-include-with-the-composer-removal` records the ordering.
No `contracts/stencils` test fills the chair stencil, so the include needs no other stencil-test edit.

### seat-runner-gate-once-own-card (MEDIUM)

The `shedfake.SeatRunner` gate-once change, with its `TestSeatRunner_LastEntryRepeatsAndFnsOverride` rows, moves from the row-switch card into the new card 6, `seat-runner-gate-once`, ahead of the row switch.
Card 7 lists `internal/testkit/shedfake#SeatRunner` under `Uses`.

### seat-runner-gate-once-double-eval-and-blast (MEDIUM)

The contract in card 6 now evaluates the gate only when the answered chair result carries no `GateOutcome` of its own.
`TestSeatRunner_LastEntryRepeatsAndFnsOverride` gains a row for a `RunFn` that returns its own outcome, which leaves the closure uncalled.
In card 7, the `RunFn` in `parentreview_integration_test.go` returns the chair result carrying the `GateOutcome` its script built, so the gate is evaluated once.
`newLoomSeats` returns no outcome, so the fake evaluates Discussion-Write's own gate.
A grep shows `TestMultiLLMEntry_ComposedTable` is the only consumer that scripts a gated table with a gate-free done chair (`gates` with `plan`).
Card 6 targets it: its scripted chair result now carries a passing outcome, so it stays a table-shape test.
The ImpactSummary names this, and the `shedadapters` results that already carry `Gate` are unaffected.

### advisors-mapping-refusal-way-forward (MEDIUM)

Confirmed: `ModelSpecList.UnmarshalYAML` returns `modelSpecShapeError`, whose message ends with `modelSpecListWayForward`, the non-empty-list text.
In card 3, `LoadConfig` now returns the key, the line and the advisor way forward when the shape error's key is `discussion_advisors`.
The mapping row of `TestLoadConfig_Refuses` asserts the key and the advisor way forward with no entry index, and the refusal-spec row's way forward matches.

### d4-deviations-unrecorded (MEDIUM): resolve-at-load half

`LoadConfig` already reads `burler.yaml` from its base directory for the fan keys.
`wire` loads both the loom config and `modelspec.LoadRegistry` from `anchorPath`.
So card 3 now resolves at load, as D4 and the acceptance criterion require.
When the list holds entries, `LoadConfig` loads the registry through `modelspec.LoadRegistry(baseDir)`.
`validateAdvisorSpecs` takes the registry and refuses an unknown alias, naming the key, the index and the way forward; a new refusal row covers `["sonnet[high]", "ghost"]` → `entry 2`.
An unreadable registry is refused with its own way forward and its own refusal-spec row.
`DiscussionTable` still resolves each entry, because it needs the model fields and takes a `Config` that may not come through `LoadConfig`.
The Shared Decision is renamed `resolve-at-load-and-in-the-builder`.

### seatengine-resigns-vs-additive-constraint (MEDIUM)

Card 1 now re-signs nothing.
`validateValues` keeps its signature: `Table.Validate` refuses a reserved listed name and hands `validateValues` a copy of the values without the blank listed names.
A new `composeSeatPrompt` carries the optional list.
`composePrompt` keeps its signature and delegates with no optional names, so its existing test calls are unchanged.
`startSeat` and `Prompts` call `composeSeatPrompt`.
In card 2, `MultiLLMProducer` keeps its `table` field and gains `tables` and `prepareFreshSpawn` beside it.
`NewMultiLLMProducer` is unchanged, and `Call` reads the source when one is set.
After this change `composePrompt` has only test callers.
It is kept because the Constraints forbid removing or re-signing what ships while `burler-multi-llm` edits the package; the constraint wins over the delete-unused rule, per the constraints clause of the explore instruction.

### stale-discussionspec-references (LOW)

Each cited line was confirmed.
Card 8 now edits `internal/loomengine/review.go` (file header), `internal/loomengine#PlanSpec` and `internal/shedadapters#SpecSource`, re-pointing or dropping each `DiscussionSpec` reference, beside `mergeresolve/spec.go`.
Card 7 edits `internal/loomshed#discussionWrite` and `discussionWrite.Call`, whose docs name the wrapped producer as a `*MultiLLMProducer`.
Its `engineMinimalConfig` comment edit now also names the multi-LLM producer.
`docs/benchmarks/test-redundancy.md` also names `DiscussionSpec`, but it is a measurement report and is left as it is.

### reconcile-test-duplicates-mechanism (LOW)

`plan:internal/loomengine#TestConfig_DiscussionAdvisorsKeptThroughReconcile` is dropped.
Card 3 targets `internal/yamlengine#TestReconcile`, with rows for an open-map key held as a scalar, as `""` and as null, beside its existing "open map carries a list whole" row.
Card 3 no longer edits the budget file.
The `test-budget-trajectory` decision is now +2 in card 5, with card 8 setting the final count.

## Disputed

None.

## Deferred

### d4-deviations-unrecorded (MEDIUM): absent-key half

An absent `discussion_advisors` key in a present `loom.yaml` loads as the template's one advisor, not zero, because `configengine.Load` fills a missing key from the template before the unmarshal.
D4 itself sets that template default to `[sonnet[high]]`.
The record's two outcomes, "absent key loads as zero advisors" and "the template default adds one advisor", cannot both hold in a file without bypassing `configengine`'s fill, which PATTERN-config-strictness makes the loader's contract.
Settling this needs a decision-record entry.
That is outside this round's write surface (the plan directory and this report), and the review names `lyx loom decision add --by parent` as the route.
I sent `ly:orch` a message with the proposed entry text.
Until it is recorded, the plan keeps the Shared Decision `absent-key-takes-template` and card 3's legacy-file row unchanged.

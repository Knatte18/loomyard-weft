# Card 1 — Replace the stencil claims table with the D1 keeps

**Delete:**
- `contracts/stencils/claims_test.go`

**Create:**
- `contracts/stencils#TestStencils_NameGoValues`
- `contracts/stencils#TestTwillStencils_HostNeutral`
- `contracts/stencils#TestStencils_MarkerPositions`
- `contracts/stencils#headingSection`

**Edit:**
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#TestLoomRubrics`
- `internal/loomengine#TestDiscussionTable`

**Uses:**
- `contracts/stencils#Registry`
- `internal/gatecli#SlotBusyExit`
- `internal/shedrun#ParkMarkerFileName`
- `internal/shuttleengine#MessageTail`
- `internal/websterengine#Report`

**Intent:**
Delete `contracts/stencils/claims_test.go` whole (D1, D2): the `wordingClaims` table, `claim`, its helpers (`wantAll`, `wantNone`, `inSection`, `joinClaims`, `droppedConceptClaims`, `hostNeutralClaims`, `focusEntryClaims`, `sectionOf`, `branchOf`), its constants, `TestStencilClaims`, `TestBranchOf` and `TestSectionOf`, and with it the `loom-discussion-spec.md` heading rows, which `internal/discussionparser`'s own tests pin.
Every row not moved below goes, whatever `why` or provenance it cites.

In the same commit, the kept rows land in three tests of `contracts/stencils`, all reading the embedded stencils through `Registry().Default(name)` and failing when a row's name is not registered:

1. `TestStencils_NameGoValues`, in a new file `contracts/stencils/govalues_test.go`, carries `//lyx:guard` directly above its `func` line, taking over `TestStencilClaims`'s marker so a card that renames a value outside `contracts/stencils` still fails its own card gate.
   It is one table, one row per stencil and value, each row a registered stencil name and the text built from the Go declaration:
   `webster-body-implementer` names `` "`" + strconv.Itoa(gatecli.SlotBusyExit) + "`" ``;
   `orch-template-role`, `shed-template-driver-notify` and `shed-template-driver-notify-watched` each name `shuttleengine.MessageTail`;
   `shed-template-driver` names `` "`" + shedrun.ParkMarkerFileName + "`" ``;
   `webster-body-implementer` names the fork report schema's keys, `status:` and `head_sha:`, each built from the `yaml` tag of `websterengine.Report`'s `Status` and `HeadSHA` fields read through `reflect` (the tag's name part before any comma, plus `:`).
   The test fails when the table is empty, when a row's stencil is unregistered, and when the stencil's text lacks the row's value; a row's failure names the stencil and the value.
2. `TestTwillStencils_HostNeutral`, in the same new file, carries `//lyx:guard` too.
   It scans every registry name with the `twill-` prefix (`Registry().Names()`), fails when that set is empty, and fails for each phrase of the host-neutral ban found in a stencil's text, matched case-insensitively, naming the stencil and the phrase.
   The ban list is the one `hostNeutralClaims` held: `decision record`, `support log`, `decision-record.md`, `support-log.md`, `_lyx/discussion`, `Step 5`, `Step 6`, `validate-discussion`, `## Interview`, `Discussion`, `validate-plan`, `00-overview.md`, `lens-review.md`, `Card Index`.
3. `TestStencils_MarkerPositions`, in `contracts/stencils/markers_test.go`, is one table of marker positions, each row anchored on markers, headings and the end of the body only:
   `webster-body-implementer`'s text ends with `{{.kickstart_pack}}\n`;
   `webster-body-implementer` holds `{{.kickstart_pack_inline}}\n\n## The sandbox rule`;
   and each section-scoped row holds its marker inside its heading's section, through `headingSection`: `{{.twill_directive}}` in `loom-template-plan`'s `## Step 2 — Explore the codebase` and in `loom-template-discussion`'s `## Step 4 — How to get answers`, `{{.decision_record_path}}` and `{{.support_log_path}}` in `loom-template-discussion`'s `## What you may write`, `{{.parent_notify}}` in `shed-template-driver`'s `## Notifying the parent`, and `{{.arbiter_command}}` in `shed-template-driver`'s `## Arbiter`.
   A missing heading fails the row naming the heading.
   `headingSection` returns the text from the line equal to the heading, or starting with the heading and a space, up to the next line starting `## `, and whether the heading exists; it is `sectionOf`'s behavior, now tested through this table alone.
   The `{{.specs_dir}}/loom/loom-discussion-spec.md` row under `## Step 5 — Write the two output files` goes (D2).
   In the same card, `TestDiscussionTable` in `internal/loomengine` adds `discussionTableSpecsDir` to its chair wants: that row named the stencil's one `{{.specs_dir}}`, and the test's existing `table.Values["specs_dir"]` check reads the value map, not the rendered prompt, so without the want removing the marker would fail no test.
   D2 names every row of this table, so it stays even where a render test also reaches the position: `TestPlanTable`'s order check reaches the plan template's `{{.twill_directive}}` row, and `renderForkPromptAndPack`'s suffix check reaches the `{{.kickstart_pack}}` row, but each of those reads a rendered prompt and fails on a render change too, while this table pins the stencil's own text.

`TestStencils_TopLevelMarkers` gains one row per stencil whose claims row named a marker that no Fill round-trip or render test reads in its real file, so removing one of those markers still fails a test:
`landing-template-describe-darn` `{decision_record_path, description_path, edit_directive, parent_branch, parent_directive, review_dir, slug, task_branch}` (its only render test fills a fixture string, not the real file),
`webster-body-verify-fix` `{friction_directive, outcome_path, plan_dir, report_path, summary_path, worktree_root}` (no render test reads `worktree_root`),
`bouncer-template-escalation` `{cause, decide_options, decision_record_path, judge_stencil_path, ledger_path, review_path, round, segment, slug, verdict_path, worktree}`
and `bouncer-fragment-decide-capped` `{slug}`, each list sorted as `stencil.TopLevelMarkers` returns it.
The stencils whose marker-naming rows go without a new row are read by a test of their real file: the arbiter, alignment and tackler prompts and the Burler halves by their render tests, `darn-template-write` by `internal/darnengine`'s `TestDarnSpec`, the shed driver's `{{.guide_path}}` by `internal/loomcli`'s `TestDriverPrompt`, the webster templates by `TestTemplates_FillRequiresEveryRequiredMarker`, `bouncer-template-parent-notice` and `bouncer-fragment-decide-open` by `internal/shedadapters`' escalation test, which asserts the rendered notice's slug and brief path and the below-cap brief's `lyx loom circling continue my-task`, `loom-template-discussion`'s `{{.specs_dir}}` by the `TestDiscussionTable` want above, the twill, webster fragment and seat stencils by their existing `TestStencils_TopLevelMarkers` rows.
`TestStencils_TopLevelMarkers`' `//testtiming:keep` reason stops naming the wording claims: "pins the exact marker set of each stencil, which the rubric test's subset check and the render tests do not".

`TestLoomRubrics` gains one assertion, scoped by a new per-row boolean `namesSpecs` in its table: true for `loom-rubric-discussion-review`, `loom-rubric-plan-review`, `loom-rubric-webster-review` and `loom-rubric-merge-review`, false for `loom-rubric-darn-review`.
The rendered text contains the `specs_dir` value it renders with, `/abs/specs/dir`, exactly when `namesSpecs` is set, the same equality shape the `publish_failure`, `webster_record` and `merge_review` checks use.
The darn-review rubric carries no `{{.specs_dir}}` today, and card 8 adds none.
Its existing assertions already fail when `publish_failure`, `webster_record` or the plan rubric's `stencils_dir` leaves its rubric, so this one assertion is what keeps a removed `{{.specs_dir}}`, the marker the claims rows named for the plan-review and webster-review rubrics, failing a test.

The file comment heading `markers_test.go` stops naming `claims_test.go`: it says the file pins the exact top-level marker set of each stencil whose markers no render test reads in its real file, and the position of each Go-filled marker a row anchors on another marker, a heading or the end of the body.
The file comment heading `rubric_test.go` drops its last line, "The rubrics' wording is pinned in claims_test.go.", and its second line also names the `specs_dir` render check; the comment sits in the file `TestLoomRubrics` lives in.

The package's top-level test count stays at its `cmd/lyx/testdata/test-budget.yaml` row: three tests leave, three arrive.

**ImpactSummary:** test-only change in `contracts/stencils` plus one added want in `internal/loomengine`'s `TestDiscussionTable`; the deleted helpers have no caller outside `claims_test.go`, and the two `//lyx:guard` tests keep the card-gate guard on the Go values `gatecli`, `shedrun`, `shuttleengine` and `websterengine` declare.

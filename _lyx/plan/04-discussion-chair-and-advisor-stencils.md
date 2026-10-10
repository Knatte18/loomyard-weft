# Card 4 — discussion-chair-and-advisor-stencils

**Create:**
- `contracts/stencils/loom/loom-template-discussion-chair.md`
- `contracts/stencils/loom/loom-template-discussion-advisor.md`
- `plan:contracts/stencils#LoomTemplateDiscussionChair` -> `var LoomTemplateDiscussionChair []byte`
- `plan:contracts/stencils#LoomTemplateDiscussionAdvisor` -> `var LoomTemplateDiscussionAdvisor []byte`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#openingStencilViolations`
- `contracts/stencils#TestParentDirectiveScans_SyntheticRows`

**Uses:**
- `contracts/stencils#SeatDirectiveChair`
- `contracts/stencils#SeatDirectiveAdvisor`
- `contracts/stencils#LoomTemplateDiscussion`
- `internal/stencil#IncludeNames`

**Intent:** Ship the advisor stencil D5 describes and a chair stencil that carries D6's content without touching the single-agent stencil, which the parent's decision keeps unchanged.
The chair stencil `loom-template-discussion-chair` opens with a header comment naming the seat engine as its only filler and the two blocks it includes, then renders, in this order, the include `{{template "loom-template-discussion"}}`, the include `{{template "seat-directive-chair"}}` and a section on how the advisors take part.
It includes the discussion stencil whole rather than copying it: the two directive markers, the pattern and friction markers, `{{.mode_rules}}`, the steps, the self-check, the gate messages and the fence all reach the chair through that include, filled from the chair's value map, so the chair's write surface and gates cannot drift from the single writer's (PATTERN-producer-pointer-rule), and the single-agent composer keeps filling the discussion stencil with no include of its own.
The advisors section keys itself to the included steps: after Step 2 the chair sends each advisor its first batch of design questions, and assigns each advisor an area when there is more than one; in autonomous mode every interview batch goes to the advisors before the chair settles it, and a pick made with no usable answer stays an auto-pick; in interactive mode the batches go where Step 4 says while the advisors supply facts, and the operator's turns reach the chair alone; `## Interview` carries each advisor answer the chair relied on under its turn, naming the advisor's strand name, and the question ledger marks a pick resting on such an answer as advisor-informed, naming the same strand; the chair reads each advisor's notes file when it exists, writes off an advisor that does not answer in reasonable time and carries on after a failure notice; the step ends exactly as Step 6 says.
No sentence of the section uses an ask-the-operator phrase without a prohibition word, since the parent-directive scan reads every registered stencil, and the interactive wording stays in `mode_rules`.
The advisor stencil `loom-template-discussion-advisor` opens with a header comment naming its markers and the seat engine as its filler, then renders `{{.parent_directive}}`, `{{.edit_directive}}`, the include `{{template "seat-directive-advisor"}}` and `{{.pattern_directive}}`, and carries `{{.slug}}`, `{{.decision_record_path}}`, `{{.support_log_path}}` and `{{.output_files}}`.
Its steps are D5's six items: read the board entry for the slug and every entry touching it, PATTERN with the background files that touch the task, and the code at module-boundary altitude; write the notes file as soon as the study is done, before any question arrives, holding the module boundaries, the PATTERN and board entries touched and any conflict seen; answer every question from the chair over the session message channel naming the evidence and a recommendation, saying so when the code does not answer; message the chair unprompted when a design message or the draft decision record conflicts with a PATTERN or board entry; write only the notes file, message only the chair and never the operator or the parent, treat everything else as read-only and never mutate git; keep answering until the chair ends the step.
It points at the chair's decision record path for the record's format and restates no section set, names no skill and never tells an agent to ask the operator.
`entries` registers both stencils after the discussion stencil; `roleOpeningStencils` keeps its `discussion` key and adds `discussion-chair` mapped to the chair stencil and `discussion-advisor` mapped to the advisor stencil, the strand roles the seat engine forms from the `discussion` prefix.
`openingStencilViolations` resolves includes: a mapped stencil whose own body lacks the marker passes when a registered stencil it includes, read through `stencil.IncludeNames`, carries it, so the chair stencil satisfies the parent-directive and edit-directive tests through its include; a subtest joins `TestParentDirectiveScans_SyntheticRows` with a stencil carrying the marker only through an include, and one whose include lacks it too.
`wordingClaims` gains a chair-stencil row whose claims pin the two includes by their template actions, the `## Interview` record and the advisor-informed ledger mark each naming the strand, and an advisor-stencil row whose claims pin: the notes file is the only file written, the chair is the only session messaged, git is never mutated, and the notes are written before questions arrive; every existing discussion-stencil claim stays.
`TestStencils_TopLevelMarkers` gains the advisor stencil's exact marker set and a row pinning that the chair stencil declares no top-level marker of its own.
The registry on-disk test, the skill-load scan, the edit-rule scan and the citation scan cover the new files without edits.

**ImpactSummary:** Two new embedded stencils, two map entries and one include-aware test helper; the discussion stencil's bytes and every other existing stencil's are untouched, so the single-agent composer, its tests and the discussion rubric test keep passing.

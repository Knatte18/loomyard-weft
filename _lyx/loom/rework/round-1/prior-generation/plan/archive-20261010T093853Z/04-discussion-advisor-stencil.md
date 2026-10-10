# Card 4 — discussion-advisor-stencil

**Create:**
- `contracts/stencils/loom/loom-template-discussion-advisor.md`
- `plan:contracts/stencils#LoomTemplateDiscussionAdvisor` -> `var LoomTemplateDiscussionAdvisor []byte`

**Edit:**
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Uses:**
- `contracts/stencils#SeatDirectiveChair`
- `contracts/stencils#SeatDirectiveAdvisor`

**Intent:** Ship the advisor stencil D5 describes and turn the discussion stencil into the chair's (D6, D10).
The advisor stencil `loom-template-discussion-advisor` opens with a header comment naming its markers and the seat engine as its filler, then renders `{{.parent_directive}}`, `{{.edit_directive}}`, the include `{{template "seat-directive-advisor"}}` and `{{.pattern_directive}}`, and carries `{{.slug}}`, `{{.decision_record_path}}`, `{{.support_log_path}}` and `{{.output_files}}`.
Its steps are D5's six items: read the board entry for the slug and every entry touching it, PATTERN with the background files that touch the task, and the code at module-boundary altitude; write the notes file as soon as the study is done, before any question arrives, holding the module boundaries, the PATTERN and board entries touched and any conflict seen; answer every question from the chair over the session message channel naming the evidence and a recommendation, saying so when the code does not answer; message the chair unprompted when a design message or the draft decision record conflicts with a PATTERN or board entry; write only the notes file, message only the chair and never the operator or the parent, treat everything else as read-only and never mutate git; keep answering until the chair ends the step.
It points at the chair's stencil for the decision record's format (PATTERN-producer-pointer-rule) and restates no section set, names no skill and never tells an agent to ask the operator.
The chair stencil keeps every claim it carries today and gains: a header comment naming the seat engine as its filler, beside `composePrompt` and `DiscussionSpec` until card 8 retires them; a step after Step 2 that sends each advisor its first batch of design questions and assigns each an area when there is more than one; in Step 3, that every interview batch goes to the advisors before the chair settles it in autonomous mode, that a pick made with no usable answer stays an auto-pick, and that in interactive mode batches go to the operator as today with advisors supplying facts and the operator's turns going to the chair alone; in Step 5, that `## Interview` carries each advisor answer the chair relied on under its turn naming the advisor's strand name, and that the question ledger marks a pick resting on such an answer as advisor-informed naming the same strand; and that the chair reads each advisor's notes file when it exists, writes off an advisor that does not answer in reasonable time and carries on after a failure notice.
The include `{{template "seat-directive-chair"}}` is not added here: until card 8, `loomengine.composePrompt` still fills the chair stencil through `stencil.FillOptional`, which refuses an include it has no entry for and never fills the block's markers, so the include lands in card 8 with that composer's removal.
Every chair-stencil addition here is prose with no marker, so the single-agent composer keeps rendering it.
The interactive wording stays in `mode_rules`, and no sentence of either stencil asks the operator.
`entries` registers the advisor stencil after the discussion stencil; `roleOpeningStencils` renames the `discussion` key to `discussion-chair` and adds `discussion-advisor` mapped to the advisor stencil, so the parent-directive and edit-directive tests cover both.
`wordingClaims` gains chair claims for the two reliance records naming the strand, and an advisor-stencil row whose claims pin: the notes file is the only file written, the chair is the only session messaged, git is never mutated, and the notes are written before questions arrive; every existing chair claim stays.
`TestStencils_TopLevelMarkers` gains the advisor stencil's exact marker set.
The registry on-disk test, the skill-load scan and the ask-operator scan cover the new file without edits.

**ImpactSummary:** One new embedded stencil and two map entries; every existing stencil's bytes other than the discussion stencil's are untouched, the discussion stencil gains marker-free prose only, so the single-agent composer and its tests keep passing, and the discussion rubric test still names the chair stencil.

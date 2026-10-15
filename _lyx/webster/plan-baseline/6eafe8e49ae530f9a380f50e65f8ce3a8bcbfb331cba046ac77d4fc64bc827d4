# Card 2 — writer-fork

**Create:**
- `contracts/stencils/twill/twill-writer-fork.md`
- `contracts/stencils/twill/twill-relaunch.md`
- `contracts/stencils#TwillWriterFork`
- `contracts/stencils#TwillRelaunch`
- `internal/twilldirective#WriterFork`
- `internal/twilldirective#writerForkStencil`
- `internal/twilldirective#relaunchStencil`
- `internal/twilldirective#fillPart`
- `internal/twilldirective#filledWriterFork`

**Edit:**
- `contracts/stencils/twill/twill-directive.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective#TestDirective_MissingValueNamesTheMarker`
- `internal/twilldirective#TestDirective_MissingStencilNamesIt`
- `internal/twilldirective/doc.go`
- `internal/loomengine#planTwillSpawnPoint`
- `internal/loomengine#planLensDirective`
- `internal/loomengine/plan.go`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine/template.yaml`
- `docs/overview.md`

**Intent:**
Add the directive's optional writer-fork part (D2, D3, D4) and turn it on for Plan-Write alone (D1); Discussion-Write keeps the two relaunch paragraphs.
No key or flag selects the old after-flow: a Plan-Write lead with lens forks on always runs fork0.

Stencils (all host-neutral; no `hostNeutralClaims` phrase in any case, so never "step 5", "step 6" or "discussion"; no `{{if}}`):

- `twill-relaunch.md` holds the two relaunch paragraphs moved verbatim out of `twill-directive.md` ("A writer relaunched after its process died before its Agent call …" and "… at or after its Agent call …"), markers `{{.format_step}}`, `{{.self_check_step}}`, `{{.self_check_command}}` and `{{.disposition_place}}`.
  It opens with a host-neutral banner comment naming its required markers, in the `twill-directive.md` banner style.
- `twill-directive.md` loses those paragraphs and gains two optional markers, each on no line of its own so an empty value leaves the bytes of the other mode unchanged: `{{.writer_fork}}` directly before "1. Read the rubric" (after the spawn point's blank line), and `{{.relaunch_rules}}` directly before "Lenses:" (after the re-prompt rule's blank line).
  Its markers become `disposition_place`, `dispute_evidence`, `final_self_check_command`, `governing_input`, `lens_list`, `relaunch_rules`, `reviewed_files`, `rubric_path`, `severity_rule`, `spawn_point` and `writer_fork`.
  Its banner comment lists the required markers and names `writer_fork` and `relaunch_rules` as optional, each filled by `Directive` with one part stencil.
- `twill-writer-fork.md` holds the fork0 procedure, fork0's prompt and its failure paths, markers `{{.explore_step}}`, `{{.input_record}}`, `{{.reviewed_files}}`, `{{.last_file}}`, `{{.files_dir}}`, `{{.format_step}}`, `{{.self_check_step}}`, `{{.self_check_command}}` and `{{.disposition_place}}`, following the benched `fork0-preamble.md` and `fork0-block.md`.
  It opens with a host-neutral banner comment naming its required markers, in the `twill-directive.md` banner style.
  Its body:
  - "In this run one fork, fork0, writes the reviewed files: you explore, fork0 thinks and writes, and you review and finish what it wrote. Do these steps in place of running `{{.format_step}}` and `{{.self_check_step}}` yourself, in this order:"
  - a. Explore per `{{.explore_step}}` until your context holds everything the reviewed files will name: every file, member, test, fake and fixture they will cite. Design none of their content, neither in a reply nor in a file.
  - b. Do step 1 of the lens review below now, reading the rubric, so fork0 and every lens fork inherit it; when you run the lens review, skip its step 1.
  - c. Spawn one fork, fork0, through the Agent tool with `subagent_type: "fork"`, never passing a `name`; its prompt is these lines, copied verbatim, and nothing else; wait for it, ending your turn while it is outstanding being allowed. The lines: "You are fork0, the writer: your context holds {{.input_record}}, the exploration and the rubric." / "Think the reviewed files through, then write them per `{{.format_step}}`, {{.last_file}} last, and run `{{.self_check_command}}` per `{{.self_check_step}}` until it passes." / "Read only what your context lacks and the reviewed files need." / "Never call the Agent tool, never write {{.disposition_place}}, and run no git command." / "Return one line as your final message: the files you wrote and that `{{.self_check_command}}` passed. Add no reasoning, design note or summary."
  - d. When fork0 has returned, read {{.reviewed_files}}, each in full with the Read tool, and take them as the files to review, not your own draft: you see them without fork0's reasoning. Run `{{.self_check_command}}` and fix what it reports until it passes.
  - "Then run the lens review below once; fork0 does not count against its one fork pass, and the writer it names is you. This session spawns fork0 at most once and the lens forks at most once: a fresh session is a new writer, and a resumed one is not."
  - One rule per D3 path, each its own bullet: fork0 returns and {{.last_file}} is missing or the self-check fails (complete or fix the files yourself per `{{.format_step}}` and `{{.self_check_step}}`, and spawn no second fork0); a fresh session started after an earlier one ended before {{.last_file}} existed (its partial files were archived; run this prompt from its first step, fork0 included); resumed in the same conversation before the fork0 call (go on from the first step not finished, fork0 included); resumed while the fork0 call was in flight (fork0 is lost and no second one is spawned: read every file in {{.files_dir}} in full, complete the files yourself as in the first path, then run the lens review once); resumed after fork0 returned and before the lens spawn call (spawn no second fork0, finish step d if unfinished, then run the lens review once); resumed at or after the lens spawn call (every lens fork is lost and none is spawned again: resume at the first of the lens review's steps from judging the findings onward not finished, with the findings already held, and record at {{.disposition_place}} that the lens review was cut short, naming each fork that had not returned as "no report"); the session ended after {{.last_file}} existed (the run counts as finished; while the session is live the gate re-prompts it on a missing lens review or its findings, and once it is not, the gate is evaluated once over the files as they stand, so a run with no recorded lens review fails; a later resume takes the matching resumed path); and a re-prompted writer, which spawns no fork0 and does step d first if it has not, then follows the re-prompt rule below.

`contracts/stencils`: embed the two new files as `TwillWriterFork` and `TwillRelaunch` (doc comments in the `TwillDirective` style) and register `twill-writer-fork` and `twill-relaunch` in `entries` after `twill-directive`, so `TestRegistry_MatchesOnDiskTree` and `stencilkit.Seed` see them.

`internal/twilldirective`:

- `WriterFork` holds the host values only the writer-fork part needs, each rendered verbatim: `ExploreStep` (the host step the writer explores in), `InputRecord` (the phrase naming fork0's input), `LastFile` (the file written last that marks the reviewed files complete, with the host's backticks) and `FilesDir` (the directory the reviewed files are written into, with backticks).
- `Values` gains `WriterFork *WriterFork`: nil asks for no writer fork.
- `fillPart` reads stencil `name` from `stencilsDir` through `stencilstore.Read`, fills it with `stencil.FillOptional(content, values, optional)` and returns it without trailing newlines; each error names the stencil (the read and fill wrapping `Directive` has today moves here).
- `Directive` builds the shared value map, then fills exactly one part: with `WriterFork` set, `twill-writer-fork` (the map plus `explore_step`, `input_record`, `last_file`, `files_dir`) into `writer_fork`; otherwise `twill-relaunch` into `relaunch_rules`.
  The filled part gets "\n\n" appended, the other part is "", and the directive is filled with both names optional.
  A part's required value left empty fails its fill with an error naming the marker and the part's stencil; a missing part stencil fails naming it.
- The package doc says the writer-fork part exists, that a host asks for it through `Values.WriterFork`, and that it replaces the relaunch paragraphs, which a host without a writer fork keeps.

Plan-Write host:

- `planLensDirective` sets `WriterFork` to `ExploreStep: "Step 2"`, `InputRecord: "the decision record"`, `LastFile` the backticked `planparser.PlanOverview(anchorPath)` and `FilesDir` the backticked `planparser.PlanDir(anchorPath)`; `discussionLensDirective` leaves it nil.
- `planTwillSpawnPoint` keeps its opening-line override, its second sentence becomes the spawn timing "Once fork0's plan is read and `lyx loom validate-plan` passes per the fork0 steps below, run this procedure once, before you end your turn.", and the Plan-Write added line now says "every lens fork's prompt" (D5's added line otherwise unchanged).
  The "in place of" sentence lives in the stencil part, not here (D4).
- `plan.go`'s file comment describes the fork0 flow: the lead explores, spawns fork0 to write the plan, reads it in full, self-checks, then spawns the lens forks and judges their findings as a critical fixer; the forks hold the exploration and the plan but not fork0's reasoning.

Tests (no new top-level test; every case is a row or assertion of an existing test):

- `filledWriterFork` returns a `WriterFork` with `EXPLORE-STEP`, `INPUT-RECORD`, `LAST-FILE` and `FILES-DIR`.
- `TestDirective_RendersEveryValueVerbatim` becomes two rows over `filledValues()`: without a writer fork it wants "runs `lyx check --now` per `STEP-CHECK` until it passes", "STEP-FORMAT" and "died before its Agent call still spawns once", and no "fork0"; with `filledWriterFork()` it wants each writer-fork value, "Spawn one fork, fork0", "LAST-FILE last", "every file in FILES-DIR", wants "Spawn one fork, fork0" after "SPAWN-POINT-SENTENCE" and before "1. Read the rubric", and no "died before its Agent call".
  Both rows keep the no-marker, no-banner and no-trailing-newline checks and add no "\n\n\n".
- `TestDirective_MissingValueNamesTheMarker` gains rows, each with `filledWriterFork()` and one field blanked, naming `explore_step`, `input_record`, `last_file`, `files_dir` and `format_step` (the writer-fork part's fill), and rows without a writer fork with `SelfCheckCommand` or `SelfCheckStep` blank naming `self_check_command` or `self_check_step` (the relaunch part's fill).
- `TestDirective_MissingStencilNamesIt` becomes rows: the directive removed; `twill-writer-fork` removed with a writer fork; `twill-relaunch` removed without one; each error names the removed stencil.
  Two more rows render cleanly with the other part's stencil removed (`twill-relaunch` removed with a writer fork, `twill-writer-fork` removed without one), so a `Directive` that reads both parts fails.
- `TestStencils_TopLevelMarkers`: the `twill-directive` row gets the new set; new rows `twill-writer-fork` and `twill-relaunch` pin theirs.
- `wordingClaims`: the relaunch claims ("died before its Agent call still spawns once", "never spawns again", `{{.format_step}}`, `{{.self_check_step}}`, `{{.self_check_command}}`) move from the `twill-directive.md` row to a new `twill-relaunch.md` row joined with `hostNeutralClaims()`; the directive row adds `{{.writer_fork}}` and `{{.relaunch_rules}}`; a new `twill-writer-fork.md` row joined with `hostNeutralClaims()` claims `subagent_type: "fork"`, "never passing a `name`", "Design none of their content", "skip its step 1", "each in full with the Read tool", "without fork0's reasoning", "Add no reasoning", "Never call the Agent tool", "run no git command", "Read only what your context lacks", "fork0 does not count against its one fork pass", "spawn no second fork0", "at most once", "cut short", "no report", every one of its markers, and mustNot "a later turn of the same session".
  Each D3 path also gets one claim on a phrase only it carries, so deleting any path fails: "run this prompt from its first step, fork0 included" (fresh session), "go on from the first step not finished" (resumed before the fork0 call), "read every file in {{.files_dir}} in full" (resumed in flight), "finish step d if unfinished" (resumed after fork0 returned), "naming each fork that had not returned" (resumed at or after the lens spawn call), "the run counts as finished" (ended after {{.last_file}} existed), "while the session is live the gate re-prompts it" (its live sub-case), "evaluated once over the files as they stand" (its no-longer-live sub-case), "does step d first if it has not" (re-prompted writer), and "complete or fix the files yourself" (incomplete fork0 return).
  The implementer keeps each of these phrases verbatim in the stencil, "while the session is live the gate re-prompts it" included.
- `TestPlanTable`'s "lens review" subtest drops the "runs `lyx loom validate-plan` per `Step 5` until it passes" and "Once you have written the plan per Step 3 and Step 4 …" wants, asserts every line of `planTwillSpawnPoint`, and wants the host values where they land: "Explore per `Step 2`", "your context holds the decision record", "run `lyx loom validate-plan` per `Step 5` until it passes", "`" + overview + "` last" and "every file in `" + plan dir + "`"; the stencil-only wording stays with `wordingClaims`; asserts no "died before its Agent call"; and orders "## Step 2" < "## Lens review by forks" < "Spawn one fork, fork0" < "1. Read the rubric" < "## Step 3".
  Its "no lens key" rows already assert no "## Lens review by forks", the bytes `overview-seats` leaves.
- `TestDiscussionTable`'s "lens review" subtest asserts the lead prompt has no "fork0"; its existing "runs `lyx loom validate-discussion` per `Step 6` until it passes" want now pins that the relaunch part renders at this host.

Docs, in this commit: `template.yaml`'s `plan_lenses` comment says the lead explores, has one fork write the plan, then forks once per lens of the fan to review it before it finishes; `docs/overview.md`'s Plan-Write lens line describes the fork0 flow (explore, fork0 writes and self-checks, the lead reads the plan in full, self-checks, spawns the lens forks, judges as a critical fixer and records `lens-review.md`).

The Plan-Write row's gate, its outputs and its fork authorization are unchanged: `ValidatePlanLensReview` still fails a run with lens forks on and no `lens-review.md` with `lens-review-missing`, pinned by `internal/loomshed`'s gate test and `internal/shedrecipe`'s Plan-Write entry test.

**ImpactSummary:** Adds two registered twill stencils and an optional writer-fork part to the directive renderer, switched on by the Plan-Write host alone; the Discussion-Write prompt keeps its relaunch paragraphs, and only the twilldirective, loomengine and contracts/stencils tests move.

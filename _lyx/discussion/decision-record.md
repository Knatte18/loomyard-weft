# Decision record: Twill fork0, strict critical fixer, branch-coverage lens

## Goal

Build the Twill design the benches settled for Plan-Write (operator, 2026-10-14; `docs/research/twill-rationale-bench.md`, "Outcome"):
the Plan-Write session explores, one fork (fork0) writes the plan, the session reads the finished plan without fork0's reasoning, spawns the lens forks and judges their findings as a strict critical fixer.
Add a `branch-coverage` lens to the `twill-plan` fan, and close two more lens blind spots the benches found: an unregistered table entry, and a check against the wrong parent directory.

## Scope

In:

- fork0 flow for Plan-Write whenever its lens forks are on, replacing today's "writer writes, then forks lenses" flow.
- The critical-fixer judge step in the twill directive, for both hosts that render it: Plan-Write and Discussion-Write.
- `twill-plan` fan in `internal/burlerengine/template.yaml`: new `branch-coverage` lens; one sentence added to `forced-edits`; one sentence added to `criteria-coverage`.
- Docs and config comments that describe the twill flow.

Out:

- fork0 at Discussion-Write: the Discussion writer interviews and designs, and the benches measured fork0 at Plan-Write only.
- The fork0 rationale file (benched, no gain).
- Plan-Review round counts with twill (`convergence-plan`), the twill planner's efficiency audit, and the bench agent's untested ideas (merging lenses, batched fork reads, fewer `validate-plan` rounds).
- Any change to Burler's lens fans (`fan-master-preread`, `fan-prompt-tuning`) or to `burler-step-fix.md`.
- A new mechanical check on the lens review's content: the gate keeps checking only that it is recorded.

## Decisions

### D0. Built on `overview-seats`

This task builds on `overview-seats` (in Webster) and is planned against the code after it lands.
That task retires `plan_producer` and `discussion_producer` for the `plan_lenses` and `discussion_lenses` fan keys, moves Plan-Write and Discussion-Write onto the seat table with the twill directive rendered into the lead seat's prompt alone, and renames the twill host functions.
Every key, host and test this record names is meant in those terms: "Plan-Write's lens forks are on" means `plan_lenses` names a fan, and "Discussion-Write's lens forks are on" means `discussion_lenses` names one, with or without advisors.

Rationale: both tasks edit the twill hosts in `internal/loomengine` and their tests; written against today's keys, this design would contradict the landed code.

### D1. fork0 replaces the after-flow

Whenever Plan-Write's lens forks are on, the Plan-Write lead runs the fork0 flow.
No key or flag selects the old after-flow.

Rationale: the operator chose fork0 as the design to build; at level cost its lenses found more real findings across three benches with no precision loss, and keeping the after-flow selectable is a requirement nobody asked for.

### D2. The fork0 procedure

The Plan-Write lead (the master) runs, in place of doing Step 3 to Step 5 itself:

1. Explore in Step 2 until its context holds everything the plan's cards will name: files, members, tests, fakes, fixtures and glyphs.
   It designs no card, neither in a reply nor in a file.
2. Read the rubric (the lens review's step 1), so fork0 and the lens forks inherit it; the lens review does not read it again.
3. Spawn one fork, fork0, through the Agent tool as `subagent_type: "fork"` without a `name`, with the fork0 prompt and nothing else, and wait for it.
4. When fork0 returns, read the overview and every card file its Card Index names, each in full with the Read tool, and take them as the plan to review, not its own draft.
   Run the self-check (`lyx loom validate-plan`) and fix what it reports until it passes.

Then the master runs the lens review once.
fork0 does not count against the lens review's one fork pass.

fork0's prompt says: it is the plan writer and its context holds the decision record, the exploration and the rubric;
it thinks the plan through, then writes every card file and the overview last per the host's format steps, and runs the self-check until it passes;
it reads only what its context lacks and the plan needs;
it never calls the Agent tool, never writes the lens review file, and runs no git command;
its final message is one line naming the card files it wrote and that the self-check passed, with no reasoning, design note or summary.
It asks for no skill and no color, and runs on the master's model; no model key is added.

Rationale: this is the benched arm f (`docs/research/twill-fork0-bench.md`), whose prompt fragments are a working prototype; the lens forks spawn from the master, so they hold the exploration and the plan but never the writer's reasoning.

### D3. fork0 when something goes wrong

Plan-Write's one output file is the overview, and under `PATTERN-completion-signal` a run whose output files exist counts as finished however its session ended.
fork0 writes the overview before the master reads, self-checks and runs the lens review.
A session that ends comes back one of two ways: shuttle respawns a fresh session, or reed resumes the same conversation through the strand's `ResumeCmd` (`--resume` on the same session id, as after a hibernate), which keeps the conversation and loses every fork it had spawned; a gated run's strand carries no done-when list, so reed resumes it even when the overview exists.
The paths are:

- fork0 returns and the overview is missing, or the self-check fails: the master completes or fixes the plan itself per the host's format steps and self-check; it spawns no second fork0.
- The master's session ends before the overview exists and shuttle respawns a fresh session: its plan-directory rotation archives the partial card files as today; the new master runs the whole procedure from Step 1, fork0 included.
- A master resumed in the same conversation while its fork0 call was in flight has lost fork0 and spawns no second one: it reads every plan file on disk in full, completes the plan itself per the host's format steps and self-check as in the first path, then runs the lens review once.
- A master resumed in the same conversation before its fork0 call runs the procedure on from the first step it had not finished, fork0 included, since the bound still allows it one fork0.
- A master resumed in the same conversation after fork0 returned and before its lens spawn call spawns no second fork0: it finishes D2 step 4 (reading the plan files in full and the self-check) if that is unfinished, then runs the lens review once.
- A master resumed in the same conversation at or after its lens spawn call has lost every lens fork it spawned and never spawns again: it resumes at the first step from judging the findings onward that it had not finished, with the findings it already holds, and records that the lens review was cut short, naming each fork that had not returned as no report.
- The master's session ends after the overview exists: the run counts as finished.
  If the session is still live, the gate re-prompts it on `lens-review-missing` or on its plan findings.
  If it is no longer live, shuttle's liveness finalize evaluates the gate once over the files as they stand, with no re-prompt, so a run with no recorded lens review fails `lens-review-missing` instead of passing; a later resume of that session takes the matching resumed path above.
- A re-prompted master (a gate re-prompt or a parent-review reject) spawns no fork0 and edits the plan itself.
  If it has not yet read the plan per D2 step 4, it does that first.
  If it has not yet made its lens spawn call, it runs the lens review once; otherwise it spawns no lens forks.
  This re-prompt rule is host-neutral and holds at Discussion-Write too, where the same window exists between the record's write and the lens review.

Bound: each master session spawns fork0 at most once and lens forks at most once; a respawned session counts as a new master, as a respawned writer does today, and a resumed one does not.

These paths replace the twill directive's two relaunch paragraphs ("relaunched after its process died before its Agent call" and "at or after its Agent call"), which key on a single Agent call that fork0 makes ambiguous: the writer-fork part of D4 renders them in place of those paragraphs.
The re-prompt rule also replaces the directive's sentence "A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again." at every host, Discussion-Write included, so no rendered prompt carries it.
A reed resume of the same conversation is governed by the resumed paths above or, at a host with no writer fork, by the two relaunch paragraphs, never by that sentence.
A host that asks for no writer fork, Discussion-Write among them, makes one Agent call and keeps the two paragraphs as they are.

Rationale: the overview-last rule and the completion signal already decide which path a run takes, so no new state is needed; without the re-prompt rule, a session that ended between fork0 and the lens spawn would pass the gate with a lens review that ran no fork.

### D4. Where the fork0 text lives

The fork0 procedure, its failure paths and fork0's prompt are host-neutral stencil text rendered by `internal/twilldirective`, read at call time through `stencilstore` like the directive itself.
The directive gains one optional part, filled only when the host asks for a writer fork; Plan-Write asks, Discussion-Write does not, and Discussion-Write's rendered directive carries none of it.
Every host-specific noun the part needs arrives as a host value, never spelled in the stencil: at least the writer's input record, the file written last that marks the reviewed files complete, the instruction for reading the reviewed files, the format steps, the self-check step and command, and the lens review file.
The Plan-Write spawn point keeps only its override of the template's "no review logic of your own" opening line and its spawn timing; the sentence that the master runs the fork0 procedure in place of Step 3 to Step 5 is in the stencil part.

Rationale: `PATTERN-stencil-ownership` keeps prompt prose in stencils; the twill stencils' host-neutral claim (`hostNeutralClaims` in `contracts/stencils/claims_test.go`, which bars names such as "decision record", "Card Index" and "validate-plan") stays true, and a later host can opt into a writer fork with its own values.

### D5. Fork boilerplate under fork0

The fork boilerplate no longer says "the writer wrote" the files: it says the files were written and passed the self-check before the fork was spawned, which holds for both hosts.
"Review them through your lens as an outsider, as your context holds them, and never read them again" stays: under fork0 the master's Reads put the finished files in the forks' context.
Plan-Write's added line ("Report defects only, never a passed check. Give each finding's evidence as the `path:line` or glyph read.") stays.

Rationale: the bench changed this line only to say where the files came from; a host-neutral phrasing serves both hosts with one text.

### D6. The critical-fixer judge step replaces today's judge in the shared directive

The directive's step 4 ("Judge every finding with the same skepticism … {{.rejection_rule}}") becomes the critical fixer, for both hosts:

- Merge duplicates across lenses, then weigh each finding twice: its premise, against the code and the host's governing input (the decision record at Plan-Write, the board entry at Discussion-Write, each a host value), and the change it proposes.
- Give it exactly one of three dispositions, decided in this order; severity decides how a finding is reported, not whether it is weighed.
  - **Dispute**, on one of two grounds: `premise false` (the premise is false against what was read), or `every fix worse` (the premise holds, but every change that addresses it makes the reviewed files worse).
    One sound change means the finding is fixed, not disputed.
    The evidence is cited text opened in this session, of the host's evidence kinds, and what it shows against the finding.
    The kinds arrive as a host value, like the governing input.
    At Plan-Write they are a `path:line` or glyph, a `PATTERN` entry, a test or caller the change would break, or the self-check's output.
    At Discussion-Write they are the board entry, the discussion spec, the rubric, a `PATTERN` entry, the self-check's output, or a coarse code read within the writer's Step 2 bound, with no `path:line` required.
    Severity, size, cost, taste, and disagreement with the rubric or `PATTERN` never justify a dispute.
    A finding not refuted from what was read is not disputed.
    Keeping a waiver, a deferral or an unplanned test against a finding is a dispute and needs the same evidence.
  - **Fix with your own change**: the premise holds, the proposed change is wrong or harmful, and another change resolves the premise; make it and name the proposed change not made and why, with the same kinds of evidence.
  - **Fix as proposed**: otherwise; a finding with no concrete change leaves the change to the judge.
- Step 5 records one line per finding with its lens, its severity and its disposition: `disputed` with its ground and evidence, `fixed differently` with the proposed change not made, why, and what changed instead, or `fixed as proposed` with what changed; a fork that returned no report is named.

The text follows `contracts/stencils/burler/burler-step-fix.md` and the benched `critical-judge.md`, without Burler's write-surface deferral: the judge's write surface is the reviewed files, so every sound change is within reach.
The judge is the lead's step; advisors at Discussion-Write take no part in it.
The `rejection_rule` host value and Plan-Write's rejection-rule constant retire, which changes the directive's closed marker list pinned in `contracts/stencils/markers_test.go`; a phrase of the judge text that is host-specific under the host-neutral claim arrives through a host value.
The severity rules stay per host.

Rationale: the operator holds the strict prompt essential; in the fixer and fork0 benches it turned down no real finding, and its disputes and own-change fixes held up.
Discussion-Write gets it too: one judge text in the shared directive, and the same two-sided weighing already judges discussion records in Discussion-Burler-Fix.

### D7. New lens `branch-coverage` in the `twill-plan` fan

Lens text, in the fan's existing style: list every branch and exit the cards' Intents introduce or change, whether or not the decision record names it (error returns, early returns, refusals, a loop's or scan's end without a match, a default case), and for each find the planned test row that reaches it; a branch no planned row reaches is a finding.
It is added to the `twill-plan` fan after `criteria-coverage`.
It overlaps `criteria-coverage` on branches the record names, by design: `criteria-coverage` keeps them as part of its fidelity check, and `branch-coverage` exists for the ones the record does not name.

Rationale: the shared blind spot of every bench arm (KI2, KI4, KW4, KP3); `criteria-coverage` covers only branches the record names, and `row-reachability` only rows the plan already has.

### D8. `forced-edits` gains the registration sentence

One sentence: a new type, function or executor that must be registered in a table or list, such as a guard table, is a forced edit, and a registration no card lists is a finding.

Rationale: KW3 (a new executor missing from the guard table) went unflagged by every lens.

### D9. The wrong-parent-directory miss goes into `criteria-coverage`

One sentence in the `criteria-coverage` lens: check that each path, comparison or default a card states uses the operand the record names, not a near one, such as the current directory's parent where the record names the worktree's parent.
The plan rubric stays unchanged.

Rationale: the miss (J6 in `docs/research/twill-plan-bench.md`) is a card departing from what the record names, which is `criteria-coverage`'s fidelity check; the lens reaches every twill plan, where the rubric's extra text would also land in every Plan-Review round.

## Constraints

- `PATTERN-stencil-ownership`: fork0's text and the judge text live in stencils read through `stencilstore`; host values stay short phrases, not procedure text.
- `PATTERN-producer-pointer-rule`: the directive points at the host's format steps and self-check, never restating the plan format.
- `PATTERN-completion-signal`: D3 relies on it unchanged; Plan-Write's output file stays the overview alone.
- The twill stencils stay host-neutral (`hostNeutralClaims`).
- With Plan-Write's or Discussion-Write's lens forks off, the rendered prompt is byte-identical to the one `overview-seats` leaves.
- The Plan-Write row's gate is unchanged: a run with lens forks on still requires a recorded `lens-review.md`, and fork authorization stays set exactly on a lead whose lens forks are on.
- `PATTERN-test-economy`: changed stencil text extends the existing claims, markers and config tests, pinning only what must hold (see `lighter-test-rules`); a new top-level test is added only for behavior no test reaches.

## Auto-mode assumptions

- The critical judge applies to Discussion-Write as well as Plan-Write (D6), though the benches measured Plan-Write only.
- fork0 replaces the after-flow outright, with no switch back (D1).
- The wrong-parent-directory sentence goes into the `criteria-coverage` lens, not the plan rubric (D9).
- A fork0 that returns without a complete plan is finished by the master, not respawned (D3).
- This task is planned after `overview-seats` lands (D0); `ly:orch` holds the run before Plan-Write until then and merges main in.

## Open risks

- fork0 leaves the master without the writer's reasoning when it fixes findings; the benches saw no harm at Plan-Write, but no bench measured the critical judge at Discussion-Write, nor whether its design-level evidence kinds (D6), bounded by the writer's Step 2 ban on exact signatures and `file:line` citations, let it refute a finding as often as the code-level kinds do at Plan-Write.
- A hub `burler.yaml` that overrides the `twill-plan` fan keeps its own lens list and misses `branch-coverage` until it is edited; the prime hub overrides no twill fan today.
- If Plan-Write starts before `overview-seats` lands, the plan meets today's keys and hosts and conflicts with that task's cards on the twill hosts and their tests.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d3-reprompt-contradicts-directive-sentence (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- With Plan-Write's lens forks on, the rendered Plan-Write prompt carries:
  - the fork0 procedure (explore without designing, read the rubric once, spawn fork0, read the finished plan in full, self-check) and fork0's prompt;
  - each failure path of D3 as its own rule: fork0 returns an incomplete plan, the session ends before the overview exists and is respawned, a master resumed before its fork0 call, a master resumed while fork0 was in flight, a master resumed after fork0 returned and before its lens spawn call, a master resumed at or after its lens spawn call, the session ends after the overview exists (live and no longer live), and a re-prompted master, which spawns no fork0, reads the plan per D2 step 4 first if it has not, runs the lens review once if it has not made its lens spawn call, and spawns no lens forks if it has;
  - D3's paths in place of the directive's two Agent-call relaunch paragraphs, which it does not carry;
  - no copy of the directive's sentence that a gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again;
  - the critical-fixer judge step with its three dispositions and the step 5 record line;
  - the host-neutral fork boilerplate of D5;
  - `branch-coverage` among its lenses.
- With Plan-Write's lens forks off, the rendered Plan-Write prompt carries no twill text, as `overview-seats` leaves it.
- With Discussion-Write's lens forks on, with and without advisors, the rendered lead prompt carries the critical-fixer judge step, the D5 boilerplate, the re-prompt rule and the directive's two relaunch paragraphs, and neither a fork0 part nor the directive's sentence that a gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again; no advisor prompt carries any of it.
- The rendered judge step names each host's dispute evidence kinds from D6: at Plan-Write the code-level kinds, at Discussion-Write the board entry, the discussion spec, the rubric, a `PATTERN` entry, the self-check's output and a coarse code read, with no `path:line` required.
- The directive renderer fails with an error naming the marker when a required host value is empty, the writer-fork values included when a host asks for a writer fork.
- The twill stencils carry no host-specific name, the writer-fork part included.
- The resolved `twill-plan` fan lists `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits` and `test-economy`; `forced-edits` carries the registration sentence and `criteria-coverage` the operand sentence.
- A Plan-Write run with lens forks on whose `lens-review.md` is absent still fails its gate with `lens-review-missing`; one with a recorded lens review passes it.
- The docs that describe the twill flow (the `twilldirective` and `loomengine` package docs, `docs/overview.md`'s twill lines and the `plan_lenses` comment in `internal/loomengine/template.yaml`) describe the fork0 flow and the critical judge.

## Notes for the plan writer

- Prototype fragments: the hub prime's git-ignored `.scratch/twill-fork0-bench/setup/` holds `fork0-preamble.md`, `fork0-block.md`, `critical-judge.md` and `wrap-f2.sh`, the splice that shows where each fragment went in the prompt.
- Today's directive: `contracts/stencils/twill/twill-directive.md`, rendered by `internal/twilldirective`; the host values sit in `internal/loomengine`, moved into seat-table values by `overview-seats`.
- Stencil tests that pin the directive: `contracts/stencils/markers_test.go`, `contracts/stencils/claims_test.go` (directive claims, `hostNeutralClaims`, the plan and discussion template placement claims).
- The `twill_directive` marker sits at the end of Step 2 in the plan template and in Step 4 in the discussion template.
- Fan tests: `internal/burlerengine/config_test.go` pins each fan's lens count and names.
- Plan-directory rotation on a fresh spawn: `NewPlanDirRotator` in `internal/loomshed/planwrite.go`.

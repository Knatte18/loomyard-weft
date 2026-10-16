# Decision record — orch-role-stencil-fixes

## Goal

Bring the hub orchestrator's role stencil, `contracts/stencils/orch/orch-template-role.md`, in line with the operator's decisions of 2026-10-16 from the stencil intent audit:
the arbiter is the orch's filter for stuck reviews, comparing related runs is the alignment agent's job, the stencil carries nothing specific to one repository, and a small PR-Gate defect has a named fixer.
Give each repository one orch-only rules file, `ORCHESTRATOR.md`, that lyx renders into the orch's role file at every delivery, and move this repository's orch-only rules out of `CLAUDE.md` into it.

## Scope

In:

- `contracts/stencils/orch/orch-template-role.md`: the rewrites and cuts of D1 to D5, including the reworded design-call bullet of `## Escalations from a child`, the new repository-rules section of D6, and the leading comment's marker list.
- `contracts/stencils/bouncer/bouncer-template-escalation.md`: the materials list, settle condition and step 1 of D1, and the `budget` line of D2.
- `contracts/stencils/bouncer/bouncer-fragment-decide-open.md` and `contracts/stencils/bouncer/bouncer-fragment-decide-capped.md`: the reworded lead-in of D1.
- A board note for `reed-add-parentless-placement` on where its #673 workaround removal lands (D9).
- `internal/orchengine` and `internal/orchcli`: reading `ORCHESTRATOR.md` from a told path and filling a second role-stencil marker with it at every role-file render (D6).
- `internal/orchengine/doc.go`: the role file's new section and marker, beside the `command_index` text it already has.
- A new tracked `ORCHESTRATOR.md` at this repository's root, and the matching cuts and one pointer line in `CLAUDE.md` (D7, D8).
- Tests for the behaviour under Acceptance criteria.

Out:

- `contracts/stencils/loom/loom-template-arbiter.md` and `contracts/stencils/shed/shed-template-driver.md`.
- The arbiter's code, eligibility rule and config keys; `shed-arbiter-all` owns giving batten an arbiter, and nothing here depends on it.
- The alignment agent's stencil and code.
- Scaffolding an `ORCHESTRATOR.md` for other hubs; an absent file is the normal case.

## Decisions

### D1: The arbiter is the orch's filter for a stuck review

Decision:
In the role's `## Escalations from a child`, the bullet "A hand-on of a review loop follows a fixed rule: accept unless a finding makes the design wrong, and hand a blocker that remains after round 3 to the operator" goes.
In its place:

- A review escalation whose stop report names an arbiter attempt whose `envelope.json` carries verdict `hand-on` and an empty `cause`, the arbiter agent's own verdict, goes to a one-shot fork.
  The fork is handed the escalation brief's path and the path of that attempt's `envelope.json`, which sits in the attempt directory beside the verdict file the stop report names.
  The fork answers the envelope's `question` from recorded operator decisions only.
  A recorded operator decision is one of:
  a passage of the task's board entry that the entry marks as an operator decision with a date, as `Operator decision (<date>)` does;
  a board note whose body states that it records an operator decision;
  a decision-record entry added with `--by operator` that is already in the committed decision record when the escalation arrives;
  or a Discussion answer that the support log's `## Interview` shows the operator gave.
  An `Idea` or agent-suggestion passage, an operator view not marked as a decision, a decision-record entry added with `--by arbiter` or `--by parent`, a writer's auto-pick, and an `## Auto-mode assumptions` entry never count.
  `--by` is an unverified label: `internal/loomcli/decision.go` restricts only `--by arbiter`, so a `--by operator` entry written after the escalation arrives never counts, and the fork records only with `--by parent`.
  It follows the brief's steps to record and resume; when the recorded decisions do not answer the question, it records nothing and reports back, and the orch brings the question to the operator.
- The fork falls under the open-fork rules of `## Investigating a stop`: a repeat notice for the slug starts no second fork, and the slug is named in the orch note while its fork is open.
- The role restates no rule for deciding a stuck review: the arbiter's per-cause rule in `loom-template-arbiter.md` is the only one.

The escalation brief changes to match:

- Its materials list gains the arbiter attempt's `envelope.json`, when the parent names one, read first,
  and the support log in the decision record's directory, read when the definition below needs its `## Interview`.
- Its settle condition becomes: settle only when a recorded operator decision answers the envelope's question directly, since the arbiter already judged the review materials by its rule.
  The brief states D1's definition of a recorded operator decision, including the entries that never count and the rule that `--by` is an unverified label, so a `--by operator` entry counts only when it is in the committed decision record before the escalation.
  Never settle a question whose answer widens the task's scope, and never accept over an open design-wrong finding unless a recorded operator decision covers that finding.
  Otherwise record nothing and report back to the parent session, which asks the operator.
- Its `## What to do` step 1 records the operator decision the fork applied, citing where it is recorded, with `lyx loom decision add … --by parent`, instead of a design call the fork makes itself.
  The fork never records with `--by operator`.
  Without a decision record, the fork names the applied decision and its source in its report.
- The decide fragments `bouncer-fragment-decide-open.md` and `bouncer-fragment-decide-capped.md` reword their lead-in "Record your decision on the segment" to name the applied operator decision as the ground for the option the fork runs.

Bound: the fork may run only the brief's `decide_options` (`lyx loom circling accept` or `continue`) and `lyx loom decision add --by parent`, then `lyx loom resume`.
It settles only when a recorded operator decision, as D1 defines it, answers the envelope's question directly; an `Idea` or agent-suggestion passage, an unmarked operator view, a `--by arbiter` or `--by parent` entry, a `--by operator` entry written after the escalation arrived, an auto-pick and an Auto-mode assumption never count.
It never settles a question whose answer widens the task's scope, and it never accepts over an open design-wrong finding unless a recorded operator decision covers that finding.
In every other case it records nothing and reports back.

The section's bullet "A design call the child cannot make: decide it and record it with `lyx loom decision add`" is reworded to cover only an escalation that is not a review escalation; a review escalation never takes it and follows D1 or D2.
The `lyx loom circling continue` / `accept` bullet and the other mechanics bullets of the role section stay.

Rationale:
Operator decision of 2026-10-16.
The removed bullet duplicated the arbiter's rule with a fixed round 3 that ignores the configurable hard cap, and both the role and the brief ignored that the arbiter already applied its rule.
Bouncer renders the brief before the arbiter runs, so the brief cannot name the envelope's path; the orch hands it to the fork, and the brief says to read it.
The arbiter verb also concludes `hand-on` itself, with a non-empty `cause` (`timeout`, `died`, `missing`, `unparseable`, `precondition`) and a synthetic question, when the agent left no sound verdict; the empty `cause` in the attempt's `envelope.json` is the only field that tells the agent's own verdict apart, for the LLM driver and the Go driver alike.

### D2: An escalation without the arbiter agent's hand-on is a fault

Decision:
The role section gains one fallback rule, kept to one short paragraph:
a review escalation without the arbiter agent's own `hand-on` of D1 is treated as a fault and handled under `## Investigating a stop`, never decided as a design call by the orch.
The cases are cause `budget`; an arbiter refusal, which leaves no envelope; a `hand-on` carrying a `cause` (`timeout`, `died`, `missing`, `unparseable`, `precondition`); a job exit without an envelope; a failed re-launch after a `resume` verdict; and the arbiter switched off or out of spawns.
When the fault is diagnosed and acted on and a design question still stands, the orch brings that question to the operator.

The escalation brief gains one line: at cause `budget`, decide nothing and report back which input could not be read.

Bound: the rule covers only review escalations without the arbiter agent's own `hand-on`.
It adds no route and no command; the investigation ends in the actions `## Investigating a stop` already allows, taken after its re-check, or in a question to the operator.

Rationale:
Operator decision of 2026-10-16.
Such an escalation means a review that did not parse, a record that could not be read, or an arbiter that failed or was off: a stop to diagnose, not a design question for the orch.
The driver stencil reports an arbiter refusal at an `awaiting` stop with no claim of failure, and takes its hand-on path for a verb-concluded `hand-on`; it records what happened, and the orch's role classifies it, so the wordings do not conflict.

### D3: Comparing related runs is the alignment agent's job

Decision:
In `## Investigating a stop`, the sentence "Read both Discussions yourself at parent-review …, pause both after Plan-Write … to compare the plans before Webster, and review both PRs side by side at PR-Gate" goes.
"Work that spans runs stays with you: comparing plans across runs, comparing PRs that touch the same packages, and deciding when held runs go on" shrinks to the orch deciding when held runs go on.
"Two runs with related work never talk to each other; you align them" survives only in the `## Escalations from a child` bullet on an escalation naming a peer.
Each peer escalation is first classified by D1 and D2.
A peer escalation that is a D2 fault is investigated as a stop.
The peers' D1 hand-on escalations go to one joint one-shot fork that reads both decision records, under D1's settle rule and bound, with each slug named in the orch note as an open fork.
A peer's hand-on escalation that arrives while the first fork is open waits for that fork's report and its re-check; only then does it go to a fresh fork, under the same rule and bound, that also reads the first fork's finished outcome.

Rationale:
Operator decision of 2026-10-16.
The alignment agent already checks overlap with every run in flight and with the parent's landed commits at parent-review and PR-Gate, and escalates a real design conflict.
Plans follow from the records alignment compared, and a code-level clash surfaces at Merge-In and Merge-Review.

### D4: The PR-Gate reject names who fixes a small defect

Decision:
In `## PR-Gate`, "A small defect in that goal is fixed on the task branch" becomes: a small defect in that goal is fixed by the run's own agents through the reject, since `lyx loom reject` sends the findings to the recipe's rework row (`PR-Rework` in loom, a fresh `Darn-Write` in darn).

Rationale:
The old sentence named no fixer while `## Fixing a run` forbids the orch to edit or commit in either worktree of a task pair.

### D5: No repository specifics in the role stencil

Decision:

- `## After landing`'s `landed` bullet loses its tail "and, while the hub's landing `deploy` key is empty, run `update-plugins.sh` by hand".
- The bullet "Only while the landing `deploy` key is empty, at every wake, … run `update-plugins.sh` when `origin/main` holds a commit `origin/prod` lacks" goes.
- `## Where a finding goes`: a workaround the orch applies by hand is written down, with the issue number, in the repository's `ORCHESTRATOR.md` when only the orch applies it, and in the repository's `CLAUDE.md` only when every agent needs it; it is removed with the fix.
- `## Scratch and notes`: durable orch-only rules go in the repository's `ORCHESTRATOR.md`.

The rest of the stencil was read against the rule and holds no repository-specific text: the loom-only rule, the sleep check and the small-task `run_settings` policy are hub- and operator-wide.

Rationale:
Operator decision of 2026-10-16: the orch stencil carries nothing specific to one repository.
`update-plugins.sh` and `origin/prod` are this repository's; this hub sets `deploy`, so the lines are dormant here and wrong elsewhere.
`ORCHESTRATOR.md` is a lyx convention every hub may use, so naming it in the role is not repository-specific.

### D6: `ORCHESTRATOR.md` is rendered into the orch's role file at every delivery

Decision:

- The file is `ORCHESTRATOR.md` at the root of the prime worktree, beside `CLAUDE.md`, tracked in git.
- The role stencil gains a last section, `## This repository`, holding a second top-level marker that the role-file render fills.
  The section says the text adds this repository's orch rules to the role and overrides none of it; a rule there that contradicts the role is raised to the operator, and the role is followed meanwhile.
- The marker is filled with the file's content as it stands in the prime worktree at render time.
  An absent file, or one holding only whitespace, fills it with one line saying this repository has no `ORCHESTRATOR.md`.
  Any other read failure fails the render the way a failing command index does, naming the file and the way forward (fix or remove it): `start` refuses, the watcher's cycle aborts back to idle with the reason, and `resume-context` fails and writes no mark.
- Every role-file render reads the file: `start`, the watcher's render after `/clear`, and `resume-context` after a compaction.
  An edit to the file applies from the next delivery, as a stencil edit does.
- A role stencil without the new marker fails the render naming the stencil and the same way forward the `command_index` refusal gives, so an operator override never silently drops the repository's rules.
- `orchcli` joins the path once from the prime worktree's root and hands it to `orchengine` as a told path; `orchengine` derives no path (`PATTERN-told-geometry`, `PATTERN-cwd-resolution`).
  The file is a repository document like `CLAUDE.md`, not lyx state, so `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state` do not apply.

Bound: the render reads one fixed path in the prime, never a task pair's copy, and inlines it with no size cap; it adds no verb or flag.
The file can carry any text into the orch's role file at every delivery, and it reaches the prime through any landing, including one the alignment agent lands with no orch turn.
The only guard against a rule that contradicts the role is the section's own text, which tells the orch to follow the role and raise the conflict to the operator.

Rationale:
Operator decision of 2026-10-16: `ORCHESTRATOR.md` is the orch-only counterpart of `CLAUDE.md`, rendered into the orch's prompt the way the command index is inlined today, so it reaches the orch at every context cycle on top of its general role.
Claude Code does not load `ORCHESTRATOR.md` on its own, so no other agent spends context on it.
The file is tracked and changes only through a commit, the same trust as `CLAUDE.md`.

### D7: This repository's orch-only rules move from `CLAUDE.md` to `ORCHESTRATOR.md`

Decision:
`CLAUDE.md` keeps only what every agent in the repository must have.
These move to a new `ORCHESTRATOR.md` at the repository root, worded as rules for the orch:

- From `## Production`, the orch's standing exception, worded as an exception to `CLAUDE.md`'s Production rule: after its own pushed fix, on a clean `main`, the orch runs `update-plugins.sh` itself and says so in its report.
  The sentences that stay in `CLAUDE.md` are reworded so "the second standing route" no longer leans on the moved one.
- From `## Watching runs`, the orch-directed sentences: the operator watches each run in its own terminal window, never in VS Code, and VS Code opens only when the operator asks, through `lyx ide` or `batten.yaml`'s `open_ide`.
  The section's description of batten's `Open-Window` row is cut, since `internal/battenshed/doc.go` and the role's run loop already carry it.
- `## Workarounds` with the #673 entry, removed with the fix of #673 as D9 assigns it.

Every other `CLAUDE.md` section stays.

Rationale:
Operator decision of 2026-10-16: instructions about the orch's behaviour go to the orch only, and the #673 workaround belongs in `ORCHESTRATOR.md`.
The moved items instruct only the orch; no spawned role's stencil runs `lyx reed add` without `--parent`.
`reed-add-parentless-placement` fixes #673 and removes the workaround from `CLAUDE.md` in the same change.

### D8: `CLAUDE.md` points at `ORCHESTRATOR.md` for orch-only notes

Decision:
`CLAUDE.md`'s `## Notes go in git, not file-memory` gains one line: a rule or workaround only the orch needs goes in `ORCHESTRATOR.md`, never in `CLAUDE.md`.

Rationale:
Any agent the orch briefs to write a durable note reads `CLAUDE.md`, so the routing is stated where every agent sees it, or orch-only notes drift back into `CLAUDE.md`.

### D9: The #673 entry's removal has an owner in either landing order

Decision:

- When `reed-add-parentless-placement` lands first: this run's Merge-Review perch, which measures each merge-in against this record, checks after every merge-in whether the parent's `CLAUDE.md` still holds the #673 workaround.
  If it does not, a #673 entry in this task's `ORCHESTRATOR.md` is a finding, fixed by removing the entry.
  The check holds while the perch's budget, `merge_max_bounces`, is above 0; at a budget of 0 the perch settles without a review, and the entry is left to the orch's rule that a workaround is removed with its fix (D5).
- When this run lands first: a card of this run's plan adds a board note for `reed-add-parentless-placement` with `lyx board upsert`.
  The note says that its workaround removal targets `ORCHESTRATOR.md`'s #673 entry wherever the parent holds it there, and `CLAUDE.md`'s otherwise.
  The note is worded for either order, so it holds if that run lands first after all.
  That run's agents measure against its own record, not the board, so the note's reader is the orch: on that run's `landed` notice, when the parent's `ORCHESTRATOR.md` still holds the #673 entry, the orch hands a fresh agent its removal, as the rule that a workaround is removed with its fix (D5) directs.

Rationale:
Merge-In is a Go engine row that merges, verifies and writes the merge record; on a clean merge no agent resolves anything, so it cannot own the drop.
The Merge-Review perch runs after every merge-in and reads this record as its answer key, and its overlap skip does not apply, since the parent's removal touches `CLAUDE.md`, which this task also changes.
The plan writer schedules only Decisions, Scope and Acceptance criteria, so the board note is a Decision rather than an open risk.

## Constraints

- `PATTERN-stencil-ownership`: both stencils are still read at call time from the told stencils directory; `ORCHESTRATOR.md` is read at render time, never embedded.
- `PATTERN-told-geometry` and `PATTERN-cwd-resolution`: `orchengine` is told the file's absolute path; only `orchcli` builds it from the resolved location.
- `PATTERN-no-denied-recovery`: the rewritten stencil text names no command the agents' settings deny.
- `PATTERN-role-skills-typed`: the rewritten stencil text asks no agent to load a skill.
- `PATTERN-producer-pointer-rule`: the role points at the arbiter's rule and never paraphrases it.
- `PATTERN-test-economy` and `PATTERN-test-speed`: render cases extend the existing role-render tests where they fit; a new top-level test in `internal/orchengine` or `internal/orchcli` is checked against `cmd/lyx/testdata/test-budget.yaml`.
- `PATTERN-documentation-lifecycle` and CLAUDE.md "Docs land in the same commit": the marker and section are described in `internal/orchengine/doc.go` in the same commit.
- Semantic line breaks in every `.md` file touched, `ORCHESTRATOR.md` included.

## Auto-mode assumptions

- `ORCHESTRATOR.md` adds to the role and overrides nothing (D6): the operator's "on top of the orch's general role" reads as additive, and no moved rule needs an override.
- `## Terminology` stays in `CLAUDE.md` (D7): any session talking with the operator may meet those shorthands.

## Open risks

- An `ORCHESTRATOR.md` that grows large inflates the orch's role file at every cycle.
- An unreadable `ORCHESTRATOR.md` is persistent: every cycle aborts until it is fixed, while the orch's context keeps growing; the abort reason names the file.
- A render that reads the file while batten's Land row pulls the prime may see it missing and render the placeholder for that one delivery.
- A task branch can change `ORCHESTRATOR.md`, and the change reaches the orch at its first delivery after the landing pulls the prime, even for a run the alignment agent landed with no orch turn.
- Hubs whose deployed role stencil is a hand-edited override refuse to start the orch after the deploy until the override gains the marker or is deleted.
- If this run lands before `reed-add-parentless-placement`, that run finds no #673 workaround in `CLAUDE.md`; the removal of the `ORCHESTRATOR.md` entry rests on the orch acting on D9's board note at that run's `landed` notice.
- If this run lands second with `merge_max_bounces: 0`, no Merge-Review checks for the #673 entry, and its removal rests on the orch's workaround rule (D5).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d1-board-source-no-provenance (class design, severity MEDIUM, open in rounds 3)
  - d1-operator-label-unverified (class design, severity MEDIUM, open in rounds 3)
  - role-bullet-design-call-survives-beside-d1-d2 (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The rendered role file contains no "round 3" rule, no `update-plugins.sh`, no `origin/prod`, and no instruction to read both Discussions, pause after Plan-Write to compare plans, or review PRs side by side.
- The rendered role file sends an escalation whose arbiter attempt's `envelope.json` carries verdict `hand-on` with an empty `cause` to a one-shot fork with the brief and that envelope, defers the decision rule to the arbiter, and treats every other review escalation as a fault investigated as a stop.
- The rendered role file's design-call bullet in `## Escalations from a child` covers only escalations that are not review escalations, so no rule there has the orch decide a review escalation itself.
- A `hand-on` carrying a `cause` (`timeout`, `died`, `missing`, `unparseable`, `precondition`) is treated as a fault, never sent to the settling fork.
- The rendered escalation brief lists the arbiter attempt's envelope and the support log among its materials, defines a recorded operator decision as D1 does, `--by` bound included, settles only when such a decision answers the question directly, never on a scope-widening answer or an accept over an uncovered design-wrong finding, records the applied operator decision and its source rather than a design call of its own, and decides nothing at cause `budget`.
- The PR-Gate section names the reject and the rework row as the fixer of a small defect.
- The workaround rule routes orch-only workarounds to `ORCHESTRATOR.md` and every-agent workarounds to `CLAUDE.md`.
- Render scenarios covered by tests:
  - `ORCHESTRATOR.md` present: its content appears in the rendered role file's last section.
  - Absent, and whitespace-only: the section carries the one-line "no `ORCHESTRATOR.md`" text and the render succeeds.
  - Unreadable (for example a directory at that path): the render fails naming the file.
  - A role stencil without the new marker: the render fails naming the stencil.
  - An edit to `ORCHESTRATOR.md` between two renders shows in the second render.
- Failure scenarios at the render sites covered by tests:
  - `start` refuses on an unreadable file.
  - The watcher's clear cycle aborts back to idle with a reason naming the file.
  - `resume-context` fails on an unreadable file and writes no resume mark.
- `ORCHESTRATOR.md` exists at the repository root with the Production exception and the Watching-runs sentences, and with the #673 workaround unless D9 removed it; `CLAUDE.md` no longer carries them and carries the D8 pointer line.
- When the parent's `CLAUDE.md` no longer holds the #673 workaround at a merge-in and `merge_max_bounces` is above 0, the Merge-Review perch raises any #673 entry in `ORCHESTRATOR.md` as a finding, and the landed `ORCHESTRATOR.md` carries no #673 entry.
- The board holds a note for `reed-add-parentless-placement`, worded for either landing order, saying where its #673 workaround removal lands.

## Notes for the plan writer

- The three role-file render sites are `RenderRoleFile`'s callers in `internal/orchengine/watcher.go`, `internal/orchengine/resumecontext.go` and `internal/orchcli/start.go`; a told path on `orchengine.Paths`, set in `internal/orchcli/paths.go`, reaches all three.
- `orchPaths` roots at `location.AnchorPath()`, which may be a subpath; `ORCHESTRATOR.md` belongs at the worktree root (`WorktreePath()`), beside `CLAUDE.md`.
- The role stencil's leading comment states it "carries one marker, command_index"; it changes with the new marker.
- `loom-stencil-wording` rewords the driver stencil's hand-committed-fix sentence; D4's wording (reject, then the rework row) should read the same way.

# Support log — orch-role-stencil-fixes

## Interview

Autonomous mode: no operator answered.
Each turn is a question the writer put to itself after reading the board entry and the code, with the pick it made.

### Turn 1 — scope against the board

Read the task entry and every entry citing the same stencils.
`loom-stencil-wording` cites `orch-template-role.md` only as evidence and edits `shed-template-driver.md:67`, which this task leaves alone; `webster-fork-stencil-fixes` touches no orch stencil.
`shed-arbiter-all` would give batten's lifecycle run an arbiter; batten stops reach the orch as notices under `## Investigating a stop`, not as review escalations, so D2 does not depend on it.
`orch-split` and `creel-notices` would restructure the orch later; nothing here blocks or duplicates them.
Pick: no dependency; scope is the stencils, the render path and the two root files.
The lens review later found `reed-add-parentless-placement`, which touches `CLAUDE.md`; see the last turn.

### Turn 2 — F8: does the bouncer brief or the arbiter stencil change?

First pick: only the role stencil changes, since the brief's "settle on the materials, otherwise report back" seemed to fit.
Revised in the lens review: the brief is what the fork reads, and it neither names the verdict nor stops the fork re-judging the materials, so it joins the change (D1, D2).
The arbiter stencil already holds the per-cause rule and stays.

### Turn 3 — F8 fallback wording

Pick: one rule naming the no-verdict cases and routing them to `## Investigating a stop` (D2), with no new commands.
The operator doubts the case occurs, so the rule stays short.

### Turn 4 — F9: what survives of "you align them"

Pick: the peer clause joins the existing escalation bullet on a peer run; held-run release stays with the orch (D3).

### Turn 5 — F14: where `ORCHESTRATOR.md` is inlined

The board says "into the orch's start prompt, the way `lyx help index` is inlined today".
The index is inlined into the role file, which every one-line launch, resume and reload pointer names and which is re-rendered before every delivery.
Pick: a second marker in the role stencil, filled from a told path at every role-file render (D6).

### Turn 6 — F14: missing, empty and unreadable file

Pick: absent or whitespace-only renders a one-line placeholder; any other read error fails the render as the index does; a stencil override without the marker is refused as for `command_index` (D6).

### Turn 7 — F14: which `CLAUDE.md` text is orch-only

Went through every section.
Pick: the Production exception sentence, `## Watching runs` and `## Workarounds` move; `## Terminology` and the rest stay (D7); one pointer line is added to the notes section (D8).
Narrowed in the lens review: only Watching runs' orch-directed sentences move.

### Turn 8 — PR-Gate small defect

`lyx loom reject --help` confirms the findings go to PR-Rework in loom and a fresh Darn-Write in darn.
Pick: name the reject and the rework row as the fixer (D4).

### Turn 9 — lens review

Six forks reviewed the record; every lens returned a report.

- board-fit, MEDIUM, #673 overlaps the running `reed-add-parentless-placement`: fixed; D7 moves the workaround only if `CLAUDE.md` still holds it, and Open risks names the landing order.
- board-fit, LOW, `creel-notices` reworks the watcher: rejected; no conflict, since the reload render stays watcher logic.
- board-fit, LOW, D4 and `loom-stencil-wording` F15 both reword the fixer: fixed as a plan-writer note to keep them consistent.
- premises, MEDIUM, the driver calls an `awaiting` refusal no failure while D2 calls it a fault: fixed; D2's rationale states the driver records and the orch classifies.
- premises, LOW, a hand-on may carry no question, and an exit without an envelope has no verdict: fixed; D1 falls back to the brief's question, D2 lists the exit case.
- premises, LOW, Go-driven runs: confirmed the premise; D1 names the signal for both drivers.
- premises, LOW, only the orch runs a parentless `reed add`: confirmed; added to D7's rationale.
- premises, LOW, and goal-scope, LOW, Watching runs is partly batten's design: fixed; only the orch-directed sentences move, the rest is cut as carried by `battenshed/doc.go`.
- goal-scope, MEDIUM, the brief is one of the three wordings the board names and still re-judges the materials: fixed; the brief joins the scope (D1).
- goal-scope, MEDIUM, the brief invites a design call at cause `budget`: fixed; D2 adds a `budget` line to the brief.
- goal-scope, MEDIUM, with leanness and scenarios, MEDIUM, "the repository's rule wins" has no bound and was not asked for: fixed; the file is additive and a conflict goes to the operator (D6).
- goal-scope, LOW, the rest of the stencil was not checked for repository specifics: fixed; D5 says it was and why the remaining lines are hub-wide.
- goal-scope, LOW, Scope In left out D5's cuts: fixed.
- goal-scope, LOW, the `shed-arbiter-all` cross-reference: no change needed.
- scenarios, MEDIUM, D2 gives no end state for a design question behind a faulted arbiter: fixed; the orch brings it to the operator.
- scenarios, MEDIUM, no signal tells with-verdict from without: fixed; the signal is a `hand-on` verdict in the stop report, and a failed re-launch after `resume` is listed under D2.
- scenarios, MEDIUM, acceptance criteria miss the render-site failures: fixed; `start`, the watcher's cycle and `resume-context` are covered.
- scenarios, LOW, a persistent read failure, and a read during a prime pull: fixed as Open risks.
- scenarios, LOW, the D1 fork is not tied to the open-fork rules: fixed in D1.
- scenarios, LOW, a late peer escalation: fixed in D3.
- leanness, LOW, the brief-unchanged point stated three times: moot, since the brief now changes.
- leanness, LOW, Auto-mode bullets repeat decisions: fixed; two bullets with their reasons remain.
- leanness, LOW, Open risks repeat D6: fixed; the override risk now states the deploy consequence only.
- leanness, LOW, D6 rationale repeats its bullets: fixed.
- leanness, LOW, the edit-directive and own-board-entry constraints ask for nothing: fixed; cut.
- leanness, LOW, the generic gate-test criterion: fixed; cut.
- leanness, LOW, the stencil-deploy note directs nothing: fixed; cut.
- leanness, LOW, the Production exception must say it is an exception to `CLAUDE.md`: fixed in D7.
- pattern-fit, LOW, durable-vs-ephemeral not named: fixed in D6.
- pattern-fit, LOW, refusals should name a way forward: fixed in D6.
- pattern-fit, LOW, `cliwire-sole-wiring`: rejected; `lyx orch` is prime-only and already builds its paths from `lyxcwd.Location`.
- pattern-fit, LOW, no new PATTERN entry: agreed, no change.
- pattern-fit, LOW, the edit-between-renders scenario may share a test: no change; scenarios name behaviour, not tests.

## Rejected alternatives

- Leaving `bouncer-template-escalation.md` unchanged: the fork reads it, and it would keep a second decision rule and omit the verdict.
- Keeping a short orch-side review rule beside the arbiter's: the operator decided the arbiter's rule is the filter, and a second wording drifts as the fixed round 3 did.
- Letting `ORCHESTRATOR.md` override the role: unasked for, and it would make every role guard bypassable by one tracked file.
- Rendering `ORCHESTRATOR.md` into the one-line start pointer: pointers must stay one line for shuttle's Send, and they only name the role file.
- Rendering it only at `start`: it would not survive a context cycle, which the operator required.
- Reading `ORCHESTRATOR.md` from the anchor path: an anchored subpath may not hold the repository's root files; the worktree root sits beside `CLAUDE.md`.
- An optional marker that renders nothing when an override lacks it: an override would silently drop the repository's rules.
- Silently skipping an unreadable file: the orch would run without rules the operator wrote, with no sign.
- Moving all of `## Watching runs`: its batten description is design that `battenshed/doc.go` already carries.
- Leaving #673 in `CLAUDE.md` for the reed task to remove: the operator decided it belongs in `ORCHESTRATOR.md`.
- Moving `## Terminology` to `ORCHESTRATOR.md`: other sessions talking with the operator meet the shorthands too.
- Scaffolding an empty `ORCHESTRATOR.md` in every hub: nobody asked for it, and an absent file is handled.

## Review rounds

### Round 1

- F1, BLOCKING, D1's `hand-on` signal also caught the arbiter verb's own hand-ons (`timeout`, `died`, `missing`, `unparseable`, `precondition`): fixed; the signal is a `hand-on` with an empty `cause` in the attempt's `envelope.json`, a `hand-on` with a cause is a D2 fault, and D1's fallback to the brief's question is dropped, since the agent's `hand-on` requires a question.
- F2, MEDIUM, D1's fork had no bound: fixed; D1 gains a Bound, and the brief's settle condition mirrors its scope and design-wrong limits.
- F3, MEDIUM, D3's peer forks were not tied to D1 and D2: fixed; each peer escalation is classified first, and a late peer's fork waits for the first fork's re-checked report.
- F4, MEDIUM, the #673 move missed a landing after plan time: fixed; the condition holds through Merge-In, and Open risks names the board-note follow-up.
- F5, LOW, D6's Bound overstated the no-override guarantee: fixed; it names what the file can carry and its only guard.
- F6, NIT, deliberation narrative in D2: moved to Turn 3; D2 keeps the one-paragraph constraint.

### Round 2

- F1, MEDIUM, D7's #673 drop named a Merge-In resolution that never runs on a clean merge: fixed; new D9 gives the Merge-Review perch the check after every merge-in, and turns the board-note follow-up into a scheduled card, worded for either landing order.
- F2, MEDIUM, D1's recorded operator decision also admitted arbiter, parent and auto-mode calls: fixed; D1 defines it, names what never counts, and the brief and Bound mirror the definition.
- F3, LOW, the brief's step 1 still asked for the fork's own design calls: fixed; D1 rewords step 1 to record the applied operator decision and its source.

### Round 3

- F1, MEDIUM, a `--by operator` entry counted though `--by` is an unverified label: fixed; such an entry counts only when it is in the committed record before the escalation, and the fork records only with `--by parent`.
- F2, MEDIUM, the board entry and notes counted wholesale: fixed; only a dated passage the entry marks as an operator decision, or a note stating it records one, counts, and `Idea` and agent-suggestion passages and unmarked operator views never do.
- F3, LOW, the support log was a source the brief never handed the fork: fixed; the brief's materials gain it.
- F4, MEDIUM, the role's design-call bullet let the orch decide a review escalation: fixed; D1 rewords it to non-review escalations, and Scope and Acceptance criteria name it.
- F5, LOW, the decide fragments framed the choice as the fork's own: fixed; D1 rewords their lead-in, and Scope lists both files.
- F6, LOW, D9's perch check ignored `merge_max_bounces: 0`: fixed; D9 and the criterion bound it to a budget above 0, and Open risks names the budget-0 case.
- F7, LOW, D9's lands-first note had no reader: fixed; the orch acts on it at that run's `landed` notice.

## Question ledger

- Q1 (auto-pick): Does this task depend on `shed-arbiter-all`, `loom-stencil-wording` or `orch-split`? — No; Turn 1.
- Q2 (auto-pick): Does the bouncer brief change? — Yes, its materials, settle condition, step 1 and `budget` line; D1, D2.
- Q3 (auto-pick): Fallback for an escalation without a `hand-on` verdict? — Fault, investigated as a stop, then the operator for a standing design question; D2.
- Q4 (auto-pick): What of the cross-run alignment text survives? — Held-run release and the peer-escalation fork; D3.
- Q5 (auto-pick): Where is `ORCHESTRATOR.md` inlined? — The role file, through a second required marker; D6.
- Q6 (auto-pick): Absent, empty, unreadable file? — Placeholder, placeholder, render failure; D6.
- Q7 (auto-pick): Precedence on a conflict with the role? — None: the file is additive, and a conflict goes to the operator; D6.
- Q8 (auto-pick): Which `CLAUDE.md` text moves? — The Production exception, Watching runs' orch sentences, Workarounds if still present; D7.
- Q9 (auto-pick): Does `CLAUDE.md` point at the new file? — Yes, one line in the notes section; D8.
- Q10 (auto-pick): Who fixes a small PR-Gate defect? — The rework row, through the reject; D4.

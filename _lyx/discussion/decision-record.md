# Decision record — discussion-scope-in-lenses

## Goal

Move loom's Discussion scope check from the alignment agent into the Discussion-Review perch's lenses, send a question only the operator can settle (a plainly outdated board entry, a design conflict with another run, doubt about what the task is for) to the run's parent instead of the fixer, give the darn writer the same outdated-entry check before it starts, and rename loom's `Merge-In` row to `Parent-Align`.

## Scope

In:

- Part 1: remove the `parent-review` gate of Discussion-Write, the `lyx loom review` verbs, the parent-review round store and the alignment agent with their config keys, stencils and docs; add scope-check lenses to the Discussion-Review perch's reviewer fan, reading the board, the in-flight runs and the parent's landed commits; make the Discussion-Review Bouncer escalate a `decision`-class finding to the run's parent.
- Part 2: the outdated-entry finding goes to the parent through Part 1's route; the darn writer checks its entry against the board, the in-flight runs and the parent's landed commits before it writes its record, and asks its parent when the entry is plainly outdated.
- Part 3: rename the loom recipe row `Merge-In` to `Parent-Align` with its engine, producer, fixer mode, config keys, gate key, stencils and docs, carrying in-flight runs across.

Out:

- fabric's own merge-in: `lyx fabric merge-in`, `fabricengine`'s `MergeIn`, `MergeInAt`, `MergeInOnly`, `MergeInProgress` and `MergeVerbMergeIn`, and the prose "merge-in" for any git merge of the parent, Finalize's included.
- The Plan-Review and Code-Review perches' handling of `decision`-class findings, darn's Code-Review perch included: they keep today's fix-row notice and do not escalate.
- The `twill-discussion` fan's membership.
- A fixer `escalate` at a landing row going to the arbiter (note `landing-escalate-arbiter`).
- Moving parent messages onto creel (note `creel-agents`).
- Rewriting other board entries and the hub's orch rules that name `Merge-In`, parent review or the alignment agent, as the orch did after `fc75aef79`: notes `regression-baseline-verify` (its drift report is judged by the alignment agent), `creel-agents` (its parent-review gate and delivery stencil items), `run-verbs-shared` (`review approve`), `landing-escalate-arbiter` and `loom-beam-darns` (see Open risks).

## Decisions

### D1 — The alignment agent goes whole

Delete `internal/alignment`, its stencil `loom-template-alignment.md`, the measure fragment `loom-fragment-pr-gate-loom.md` and its wiring in `internal/loomcli`.
`DecisionNoteTitle` and its form `Decision for <slug>:` survive, moved to a package its remaining callers may import.

Rationale: the agent runs at one point only, `parent-review` (`internal/alignment/doc.go`, "The agent runs at one point of a run"); its PR-Gate point went with the PR-Gate row in `fc75aef79` (2026-10-24), whose Merge-In fixer took over the landing-time alignment.
The board entry's "keep it at pr-gate" names a row that no longer exists, so dropping the discussion point leaves the agent nothing to do.

### D2 — The parent-review gate and its verbs go

Remove the `parent-review` gate entry from Discussion-Write's `gates`, the `parent-review` gate kind from `internal/shedrecipe`'s gate registry and `Env.ParentReview`, `internal/parentreview`, `internal/loomengine`'s parent-review paths and the `lyx loom review` verb tree (`notify`, `delivered`, `approve`, `reject`).
Remove the stencils `loom-template-parent-review-brief.md`, `-delivery.md` and `-keepalive.md`, batten's `ReviewWait` seam reading the store, `lyx loom status`'s parent-review waiting note, the orch role's Parent-review section, and the Discussion-Write and twill stencils' parent-review message and carve-out text.
No text names "parent review" for the step between Discussion-Write and Discussion-Review any more.

Rationale: the operator dropped the agent at this point (board entry, 2026-10-25), and with the agent gone the gate only wakes the orch for a review the perch's lenses now make.

In-flight runs at upgrade:

- A run whose Discussion-Write waits on a parent-review round, under a driver on the new binary, passes on its next gate evaluation: the row keeps its name and its gate list holds only `discussion`; the round directory stays on disk unread.
- A run halted `blocked` at the parent-review reject cap is resumed with `lyx loom resume`; its stored stop text still names the removed `lyx loom review approve`.
- A driver still on the old binary keeps the old gate until it restarts.

### D3 — The retired config keys are dropped by reconcile

`loom.yaml`'s `alignment`, `alignment_timeout_min`, `skip_alignment` and `parent_review_wait_min` leave the template and `Config`.
Reconcile drops them from a hub or worktree file and reports each drop, and a run profile or board `run_settings` holding one is migrated at read time with a Warn per line, as `fc75aef79` did for the retired `merge_*` keys.

Rationale: a strict load refuses an unknown key, so a hub file still holding a retired key must be cleaned by the one reconciler (`PATTERN-config-layers`).

### D4 — Reports keep reading archived parent-review and alignment records

`internal/reportengine` keeps counting archived parent-review rounds and alignment attempts in its runs, wakes and timing-stops reports, through a read-only parse of the on-disk shapes (`request.json`, `delivery.json`, `verdict.json`, `alignment.json`, `attempt.json`, `verdict-<token>.yaml`) kept inside `internal/reportengine`.
Nothing writes those records any more.

Rationale: an audit window spanning the landing would otherwise lose the orch wakes and stops those rounds caused, changing earlier numbers silently.

### D5 — The perch's reviewer fan carries the scope check

`goal-scope` and `board-fit` in `internal/lenses/template.yaml` are rewritten from emphasis lenses into procedures that end in their own list, and the `review-discussion` fan gains both after its existing lenses.
`twill-discussion` keeps its members and runs the rewritten texts.

Rationale: the perch's default round-1 fan (`discussion_reviewer_fan: ["review-discussion", ""]`) holds neither lens today; both run only in the writer's own `twill-discussion` forks, where no Bouncer routes a finding.
`review-discussion` is a production fan, whose lenses are procedures ending in their own list (lens template header; `burler-cluster-fan.md`, "Report every finding your lens's procedure produces and nothing outside it"), so the emphasis texts cannot join it as written; `twill-discussion` already runs procedure lenses, so the rewrite fits it too.
Each fork has 6 tool calls beyond the reading manifest (`burler-cluster-fan.md`), which D6's reads fit.
A hub that sets `discussion_reviewer_fan` to solo rounds runs no scope check, as with any lens it turns off.

### D6 — What the two lenses read and send to the parent

`goal-scope`: read `lyx board get` for this task and every `Decision for <slug>:` note for it.
Check the record against the entry: a goal point left out, work that belongs to another entry, a direction the entry rules out, unlicensed work, or a deferral.
List, as a question for the run's parent, doubt about what the task is for.
Its list: each goal point of the entry, and whether the record meets it.

`board-fit`: read `lyx board list` and `lyx loom peers`, then up to four of `lyx board get` for a touching entry, `lyx loom peers show <slug> --record decision` or `--record plan` for a close peer, and read-only `git show` of a listed parent commit, nearest first.
List, as a question for the run's parent: the entry plainly outdated, because landed work overtook it or another board entry or run contradicts it; a design conflict with another run (both change the same behaviour or the same PATTERN entry in different ways, or a decision in one contradicts or duplicates one in the other); or a landed parent commit that already makes or contradicts one of the record's decisions.
Its list: each entry, peer and parent commit checked, and overlap, conflict or none.

Both lenses:

- A board note titled `Decision for <slug>:` for this task is a parent decision and wins over the entry body, as do the record's `Added after Discussion` entries.

- A text-only overlap (the same file or package at different places) and a shared bookkeeping line (a module-table line, a PATTERN index line) are no finding.
- A question the record's `Added after Discussion` entries or a `Decision for <slug>:` note already answers is no finding.
- An ordinary misalignment the writer could settle is an ordinary finding for the fixer.

What a fork can miss: an entry, peer or commit past `board-fit`'s four reads; its list names what it read, so a judge sees the reach.

Rationale: these are the alignment agent's escalation rules (`loom-template-alignment.md`, "When to escalate"), moved into the lenses the board entry names, within a fork's budget; the bounds keep routine overlaps from stopping a run.

### D7 — `decision` in the Discussion-Review segment means a question only the parent can settle

`loom-rubric-discussion-review.md`'s Finding class section defines `decision`, in this segment, as a question only the run's parent can settle: the cases D6 lists.
A choice the writer can make is never `decision`, and neither is a question an `Added after Discussion` entry or a `Decision for <slug>:` note already answers; a writer's Open risks line that such an answer settles is a `consistency` finding whose fix drops the line.
The section's "Class never changes whether a finding is fixed" sentence is corrected: a `decision` finding goes to the parent, never to the fixer.

Rationale: the generic meaning, "a choice left open that the author must make" (`burler-review-format.md`), lets a reviewer file a writer's choice as `decision`, which would stop the run for nothing once D8 escalates the class.

### D8 — The Discussion-Review Bouncer escalates a `decision` finding to the parent

A Bouncer row told to escalate decisions, by a row config key set on loom's Discussion-Bouncer only, escalates a judged round to the run's parent with a new cause, `decision`, when the round's accepted review holds a `decision`-class finding at any severity and no decision file is recorded for the round.

- It applies to the judged round whatever the judge ruled, before the judgment refusal over a BLOCKING finding, the soft-cap rule, the circling escalation and auto-accept, so neither a re-judge nor `discussion_auto_accept` passes it.
- A round so escalated is never a settled generation at Call entry, even with a CONVERGED verdict: a re-call returns the same Awaiting, and a recorded `continue` or `accept` takes the recorded-decision path, never the settled-entry harvest, clear or new generation, whatever `lyx loom decision add` committed meanwhile.
- It writes the escalation record and parent notice as the other causes do, and returns Awaiting.
  The brief lists each `decision` finding with its id, summary and fix, and the way forward: record the answer with `lyx loom decision add`, then `lyx loom circling continue` for a round over the amended record or `lyx loom circling accept` to proceed, then `lyx loom resume`.
- Below the hard cap it offers accept and continue; at or past it, accept only, as every cause does.
- An `accept` carries every `decision` finding of the round's review into `## Open risks`, at any severity and whatever the judge's ledger says, beside the carry-over's other entries.
- `internal/arbiterstop` refuses a `decision` escalation to the arbiter and hands it back to the parent, as it does a `budget` one.
- In that segment the Discussion-Burler-Fix row still withholds `decision` findings from the fixer, but posts no notice of its own, a deliberate change to `2e265a4fc`'s behaviour there: the Bouncer's escalation is the one parent message per round.

What it can skip or let through: nothing; it only stops a run that would have passed.
What bounds it: the class definition of D7, one escalation per judged round, and the hard cap.

Rationale: since `2e265a4fc` a `decision` finding reaches the parent by notice and leaves the fixer, but the class does not gate (`burlerengine.GatingClass` is `design`), so the judge converges past it and the run enters Plan-Write without the parent's answer; that is the "Bouncer's routing" the board entry leaves open.
The arbiter can record a decision (`--by arbiter`), so leaving it eligible would let an agent settle what only the operator may.

### D9 — The Discussion writer leaves a parent-only question open

`loom-template-discussion.md` tells the writer, in autonomous mode, not to settle a question of D6's kind itself, from its own exploration or its lens forks: it records it under `## Open risks`, naming the entry, run or commit, so the perch raises it as `decision`.
In interactive mode it asks the operator, as for any question.
The stencil also names the decision-note title form: a board note titled `Decision for <slug>:` for this task is a parent decision.

Rationale: a writer that rescopes on its own around landed work hides from the parent the very question D8 routes to it; the decision note's reader was the alignment agent, and without it the writer and the perch's lenses are the readers left.

### D10 — The darn writer checks its entry before it writes its record

`darn-template-write.md` gains a step after reading the entry and before writing the decision record, taken while the record is absent or fails `lyx loom validate-discussion` (Step 3's own condition): read `lyx board list`, every entry that touches the task and every `Decision for <slug>:` note for it, and `lyx loom peers` for the runs in flight and the parent's commits since the fork.

- When the entry is plainly outdated in D6's sense and no decision note answers it, the writer asks its parent by `SendMessage`, naming what overtook or contradicts it and asking for the answer as a `Decision for <slug>:` board note, and ends its turn to wait.
- It writes the answer into its decision record as a Decision citing the parent.
- Once the record is committed, it asks the parent to record the same answer with `lyx loom decision add`, and until the record carries that `Added after Discussion` entry it names the answer in the change description's design decisions as an unrecorded parent correction, as the stencil's scope-correction section does today.
- With no parent, it stops and reports the outdated entry.
- An ordinary misalignment it settles itself, as today.

What it can skip or let through: nothing; it only adds a wait.
What bounds it: the writer's own `writer_timeout_min`, since a turn end without every output file is held by shuttle with a hold notice to the orch (`internal/shuttleengine/doc.go`, "A turn end without every output file never ends a run"); a respawn after the timeout repeats the check and finds an answer the parent left as a decision note.

Rationale: darn has no Discussion review, and `lyx loom decision add` refuses a darn run until Darn-Write has validated and committed its record (`lyx loom decision add --help`), so the answer first comes by note or message.
Darn's Code-Review perch takes the board-entry snapshot as scope, amended only by `Added after Discussion` entries (`contracts/recipes/darn-recipe.yaml`, Code-Burler-Review fasit), so the answer must reach the record through the verb to bind the perch.

### D11 — `Merge-In` becomes `Parent-Align`

Rename, in loom's recipe and everything exclusive to that row:

- the row `Merge-In` to `Parent-Align`, with Webster's `on_done` and Finalize's `on_reroute`;
- the engine key `MergeIn` to `ParentAlign`, `landingshed`'s `MergeIn` producer, its constructor and row-name constant, and `loomshed`'s `NameMergeIn`;
- the merge fixer's mode `merge-in` to `parent-align`, and the row-named fixer report the Code-Perch reads;
- `landing.yaml`'s `merge_fix_attempts` to `parent_align_fix_attempts`;
- `gate.yaml`'s gate key `merge-in` to `parent-align`;
- recipe comments in loom and darn, stencils, specs, `pattern/` background files, `README.md`, `docs/overview.md` and package docs that name the row.

In-flight runs are carried across with `shedengine.RenameProducers`, as darn's `Darn` → `Darn-Write` rename is through `loomrecipe.DarnRetiredRows`: a status on `Merge-In` resumes on `Parent-Align`.
The Code-Perch's review-input reader takes the old row-named fixer report when the new one is absent.
`internal/reportengine` keeps classifying an archived `Merge-In` row in the merge phase and among the verify rows, as it keeps `retiredPublishRow`.
Reconcile renames a hub's `merge_fix_attempts` and `merge-in` gate key and reports each, and a run profile or `run_settings` holding the old key is migrated at read time with a Warn.

Rationale: since `fc75aef79` the row aligns the task with its parent and only starts with a merge (board entry, Part 3); the fabric verb and git-level merge-ins still only merge.
The loom recipe header's restart rule covers retired rows; a one-to-one rename has the `RenameProducers` precedent (`internal/shedengine/doc.go`, "Row renames").

## Constraints

- `PATTERN-refusal-way-forward`: the `decision` escalation's Reason and the arbiter's refusal name the way forward, and a test reaches each.
- `PATTERN-config-layers`: retired and renamed keys are handled only by reconcile and the read-time migration.
- `PATTERN-shed-recipe-registry`: D8's switch is looked up by key.
- `PATTERN-stencil-ownership`: lens text, rubric and stencils are read at call time; the lenses point at `lyx loom peers` rather than restating its output format.
- `lyx loom peers` lists each peer's row position in the loom recipe; D2 and D11 change no row position.

## Auto-mode assumptions

- D1: PR-Gate premise false; the agent goes whole.
- D4: archived records stay readable by reports.
- D5: `goal-scope` and `board-fit` become procedure lenses and join `review-discussion`.
- D8: a Discussion-Bouncer switch, any severity, replacing the fix row's notice there.
- D11: gate key and fixer mode follow the row name; the merge record keeps its name; runs are carried across.

## Open risks

- A question for the parent (D9's rule applied to this task's own entry): the entry says to keep the alignment agent at PR-Gate, a row `fc75aef79` removed; D1 removes the agent, and the parent may instead want a landing-time approval kept, such as in the Parent-Align fixer.
- `umbrella-daemon` is in flight at Discussion-Write: its entry names parent review among the orch's escalations, and it plans to move the arbiter into shed, which must keep D8's refusal of a `decision` escalation; whichever run lands second takes the other's change at Parent-Align.
- `loom-beam-darns` may run its pieces on darn; D10's check would then run inside a beam and see sibling pieces as peers, which that note's design must settle.
- The scope check now runs after the writer finishes, not before the first perch round, so a round may polish a record whose scope the parent then changes (board entry, Part 1 cost).

## Acceptance criteria

- Discussion-Write's only gate is `discussion`; a passing discussion goes to Discussion-Bouncer with no alignment spawn and no parent message.
- `lyx loom review` is no command; `internal/alignment` and `internal/parentreview` are gone.
- A run waiting in a parent-review round passes Discussion-Write on its next gate evaluation under the new binary.
- A hub `loom.yaml` holding a retired alignment or parent-review key loads after reconcile drops it with a report; a profile or `run_settings` holding one loads with a Warn.
- Reports over archived runs still count their parent-review rounds and alignment attempts, and still file archived `Merge-In` rows in the merge phase.
- `review-discussion` resolves to its existing lenses plus `goal-scope` and `board-fit`, each a procedure ending in its list with D6's reads, bounds and parent questions.
- Discussion-Review round whose accepted review holds a `decision` finding: the fixer never gets it, the fix row posts no notice, and the Bouncer returns Awaiting with cause `decision` after a CONVERGED, a CONTINUE and a CIRCLING verdict alike, and over a BLOCKING `decision` finding without a re-judge, with a brief listing the finding and the way forward.
- The same round with `discussion_auto_accept` true at the soft cap escalates `decision` instead of accepting.
- A re-call over a round escalated after a CONVERGED verdict returns the same Awaiting, with and without a `lyx loom decision add` commit in between.
- `continue` recorded on a `decision` escalation over a CONVERGED round runs another round in the same generation; `accept` settles, carries every `decision` finding, a LOW one included, into `## Open risks` and goes to Plan-Write.
- A `decision` escalation at or past the hard cap offers accept only.
- `internal/arbiterstop` refuses a `decision` escalation with `HandBackWayForward`.
- Plan-Review and Code-Review rounds with `decision` findings behave as before: fix-row notice, no escalation.
- The darn writer's stencil carries D10's check, its condition, the ask for a decision note, the wait, the later `lyx loom decision add` request and the no-parent stop; the Discussion writer's stencil carries D9; the rubric carries D7.
- The loom recipe has `Parent-Align` between Webster and Code-Bouncer and as Finalize's reroute target, built by engine `ParentAlign`; no recipe row or engine is named `Merge-In`.
- A status on `Merge-In` resumes on `Parent-Align`; a run past the row at upgrade still gives the Code-Perch its latest fixer report.
- A hub `landing.yaml` holding `merge_fix_attempts` and a `gate.yaml` holding `merge-in` are renamed by reconcile with a report, and the renamed values take effect.
- `lyx fabric merge-in` and `fabricengine`'s merge-in names are unchanged.

## Notes for the plan writer

- The escalation cause set D8 extends is checked in `internal/shedadapters/escalation.go` (`EscalationCause.valid`), `internal/shedadapters/circling.go` and `internal/arbiterstop/arbiterstop.go`, and listed in `lyx loom circling --help`.
- The re-entry rule D8 changes is `settledGeneration` in `internal/shedadapters/bouncer.go`.

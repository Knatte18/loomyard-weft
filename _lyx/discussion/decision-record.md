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
The `discussion` fan, which no default config names, keeps `goal-scope` among its emphasis lenses and runs the rewritten text too.
A hub or worktree `lenses.yaml` that defines `goal-scope` or `board-fit` keeps its own text, which beats the template as for any tuned lens; reconcile reports nothing for a name the template holds, so such a hub runs its old text in `review-discussion`, unreported, until it drops the entry.

Rationale: the perch's default round-1 fan (`discussion_reviewer_fan: ["review-discussion", ""]`) holds neither lens today; both run only in the writer's own `twill-discussion` forks, where no Bouncer routes a finding.
`review-discussion` is a production fan, whose lenses are procedures ending in their own list (lens template header; `burler-cluster-fan.md`, "Report every finding your lens's procedure produces and nothing outside it"), so the emphasis texts cannot join it as written; `twill-discussion` already runs procedure lenses, so the rewrite fits it too.
Each fork has 6 tool calls beyond the reading manifest (`burler-cluster-fan.md`); D6 gives each lens a read set of at most six calls, its decision-note reads included.
A hub that sets `discussion_reviewer_fan` to solo rounds runs no scope check, as with any lens it turns off.

### D6 — What the two lenses read and send to the parent

`goal-scope`: read `lyx board list` once, `lyx board get` for this task, and then up to four `Decision for <slug>:` notes for it, newest first.
Check the record against the entry: a goal point left out, work that belongs to another entry, a direction the entry rules out, unlicensed work, or a deferral.
List, as a question for the run's parent, doubt about what the task is for.
Its list: each goal point of the entry, and whether the record meets it; and each decision note it read, and any the list names that it could not reach.

`board-fit`: read `lyx board list` and `lyx loom peers`, then up to four of `lyx board get` for a touching entry or a `Decision for <slug>:` note for this task, `lyx loom peers show <slug> --record decision` or `--record plan` for a close peer, and read-only `git show` of a listed parent commit, nearest first.
It names this task's decision notes from the `lyx board list` titles it already holds, and spends one of its four reads on a note before raising a parent question that note's title touches.
List, as a question for the run's parent: the entry plainly outdated, because landed work overtook it or another board entry or run contradicts it; a design conflict with another run (both change the same behaviour or the same PATTERN entry in different ways, or a decision in one contradicts or duplicates one in the other); or a landed parent commit that contradicts one of the record's decisions, or already makes one in a way the writer cannot settle by citing or dropping the decision.
A landed commit that already makes a decision the writer can cite or drop is an ordinary finding.
Its list: each entry, peer and parent commit checked, and overlap, conflict or none; and each decision note it read, and any it could not reach.

Both lenses:

- A board note titled `Decision for <slug>:` for this task is a parent decision and wins over the entry body, as do the record's `Added after Discussion` entries.

- A text-only overlap (the same file or package at different places) and a shared bookkeeping line (a module-table line, a PATTERN index line) are no finding.
- A question the record's `Added after Discussion` entries or a `Decision for <slug>:` note already answers is no finding.
- An ordinary misalignment the writer could settle is an ordinary finding for the fixer.

What a fork can miss: an entry, peer, commit or decision note past its lens's reads; its list names what it read and what it could not reach, so a judge sees the reach.

Rationale: these are the alignment agent's escalation rules (`loom-template-alignment.md`, "When to escalate"), moved into the lenses the board entry names, within a fork's budget; the bounds keep routine overlaps from stopping a run.

### D7 — `decision` in the Discussion-Review segment means a question only the parent can settle

`loom-rubric-discussion-review.md`'s Finding class section defines `decision`, in this segment, as a question only the run's parent can settle: the cases D6 lists.
A choice the writer can make is never `decision`, and neither is a question an `Added after Discussion` entry or a `Decision for <slug>:` note already answers; a writer's Open risks line that such an answer settles is a `consistency` finding whose fix drops the line.
The section's "Class never changes whether a finding is fixed" sentence is corrected: a `decision` finding goes to the parent, never to the fixer.

Rationale: the generic meaning, "a choice left open that the author must make" (`burler-review-format.md`), lets a reviewer file a writer's choice as `decision`, which would stop the run for nothing once D8 escalates the class.

### D8 — The Discussion-Review Bouncer escalates a `decision` finding to the parent

A Bouncer row told to escalate decisions, by a row config key set on loom's Discussion-Bouncer only, escalates a judged round to the run's parent with a new cause, `decision`, when the round's accepted review holds a `decision`-class finding at any severity and no decision file is recorded for the round.

- It applies to the judged round whatever the judge ruled, before the judgment refusal over a BLOCKING finding, the soft-cap rule, the circling escalation and auto-accept, so neither a re-judge nor `discussion_auto_accept` passes it.
- The escalation runs ahead of the refusal for routing only: a decision recorded on a `decision` escalation still applies the judgment refusal before it acts, at the harvest and on a replay alike.
  An `accept` applies the whole refusal, with the round's `decision` findings left out of both BLOCKING checks, the review's BLOCKING-finding check and, by the entry's class, the ledger's open-BLOCKING-entry check, since the accept is the parent's answer to them; an open BLOCKING entry with no class still refuses, which the judge stencil already rules out (`bouncer-template-judge.md`: every open entry carries its class and severity).
  A `continue` applies only the dropped-key check, since it passes nothing.
  A judgment the refusal rejects takes the refusal's re-judge first: the judge outputs are archived and one counted bounce unit is spent, while the escalation record and decision file stay for the re-judged round.
  The re-judged round reaches the recorded decision again, which applies the refusal again, so the decision acts only over a judgment the refusal admits.
  A refusal that still holds once the segment's budget is reached, in a round below the hard cap, does not halt there: it retires the decision, as `retireContinue` retires a continue, and re-escalates the round with cause `decision`, its brief naming the held refusal, so the parent can record another answer, a `continue` among them.
  In a round at or past the hard cap, where a re-escalation could offer only the accept the refusal already rejects, a held refusal at the budget blocks the run at the Bouncer's row with a budget-exempt `OutputPointer.Halt` Stuck, since a budget halt persists the shed-wide budget reason and drops the producer's (`internal/shedengine/doc.go`), and its Reason names the refusal's cause and the way forward: `lyx loom resume`, whose next Call re-judges the round under the recorded accept.
  A re-judged CONTINUE or CIRCLING verdict whose ledger keeps every key of the previous round's then lets the accept pass; a re-judge the refusal still rejects halts the same way again, so each resume costs one judge spawn and the operator's resume.
  A dropped-key refusal that holds over a `continue` halts the segment at its budget, as a refused judgment does today.
  An `accept` over a re-judged CONTINUE or CIRCLING verdict passes the round as a circling accept does, carrying every BLOCKING finding the brief listed into `## Open risks`.
- A round so escalated whose decision file is absent or not yet settled is never a settled generation at Call entry, even with a CONVERGED verdict: a re-call returns the same Awaiting, and a recorded `continue` or `accept` takes the recorded-decision path, never the settled-entry harvest, clear or new generation, whatever `lyx loom decision add` committed meanwhile.
  Once its accept is settled, the round is a settled generation again, as a settled circling accept is today, and a later entry takes the existing re-settle, harvest, clear or new-generation path.
- It writes the escalation record and parent notice as the other causes do, and returns Awaiting.
  The brief lists each `decision` finding with its id, summary and fix, every BLOCKING finding of the round's review, and the judgment refusal's cause when the refusal would reject the judgment, with the note that the round is then re-judged before the recorded answer acts, that an `accept` passes the round over every BLOCKING finding the brief lists once a judgment is admitted, and in a round at or past the hard cap halts the run if the refusal still holds at the budget, and that a `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again, and the way forward: record the answer with `lyx loom decision add`, then `lyx loom circling continue` for a round over the amended record or `lyx loom circling accept` to proceed, then `lyx loom resume`.
- Below the hard cap it offers accept and continue; at or past it, accept only, as every cause does.
  A `continue` that reaches the hard-cap path at settle time re-escalates with cause `decision`, never `hard-cap`, so the round keeps its cause.
- An `accept` carries every `decision` finding of the round's review into `## Open risks`, at any severity and whatever the judge's ledger says, beside the carry-over's other entries.
  The round's first `decision` escalation writes the titles of the record's `Added after Discussion` entries at that time into a snapshot file of the round, written only when absent and moved neither by `retireContinue` nor by a re-judge, so a re-escalation of the round keeps the first snapshot, and each carried finding names the entries recorded since, so Plan-Write reads a finding beside the parent's answer to it; with none recorded since, the finding is carried bare.
- `internal/arbiterstop` refuses a `decision` escalation to the arbiter and hands it back to the parent, as it does a `budget` one.
- In that segment the Discussion-Burler-Fix row still withholds `decision` findings from the fixer, but posts no notice of its own, a deliberate change to `2e265a4fc`'s behaviour there: the Bouncer's escalation is the one parent message per round.
  The fix row is switched by its own row config key, set on loom's Discussion-Burler-Fix only, beside the Bouncer's key, and a recipe test pins both keys set together on loom's Discussion rows and neither on any other row.

What it can skip or let through: an `accept` passes the round only over a judgment the refusal admits, so it lets through what a circling accept lets through today, open findings carried into `## Open risks`, over findings the brief listed to the parent; a `continue` starts the next round only over a ledger that keeps every key of the previous round's, so the dropped-key guard holds over it.
What bounds it: the class definition of D7, which is reviewer discipline enforced by the rubric only, so a writer's choice mis-classed `decision`, or a question a decision note past a lens's reads already answers, stops the run until the parent answers; one escalation per judged round, and one more each time a refusal that holds at the budget below the hard cap retires a recorded decision; the hard cap, at or past which a held refusal halts the run; and the bounce budget the re-judge of a refused recorded decision spends.
A `continue` hands the brief's BLOCKING findings to no fixer: the perch runs the round's fixer on its own review before the Bouncer judges it, the next round's fixer fixes only that round's review, and after a CONVERGED verdict the next round's focus file is empty and the decision file carries no text.
So a listed finding is fixed again only if the next round's fresh reviewers re-raise it from the artifact, a bound accepted because the round's own fixer already had every finding but the `decision` ones.

Rationale: since `2e265a4fc` a `decision` finding reaches the parent by notice and leaves the fixer, but the class does not gate (`burlerengine.GatingClass` is `design`), so the judge converges past it and the run enters Plan-Write without the parent's answer; that is the "Bouncer's routing" the board entry leaves open.
The arbiter can record a decision (`--by arbiter`), so leaving it eligible would let an agent settle what only the operator may.

### D9 — The Discussion writer leaves a parent-only question open

`loom-template-discussion.md` tells the writer, in autonomous mode, not to settle a question of D6's kind itself, from its own exploration or its lens forks: it records it under `## Open risks`, naming the entry, run or commit, so the perch raises it as `decision`.
In interactive mode it asks the operator, as for any question.
The stencil also names the decision-note title form: a board note titled `Decision for <slug>:` for this task is a parent decision.

Rationale: a writer that rescopes on its own around landed work hides from the parent the very question D8 routes to it; the decision note's reader was the alignment agent, and without it the writer and the perch's lenses are the readers left.

### D10 — The darn writer checks its entry before it writes its record

`darn-template-write.md` gains a step after reading the entry and before writing the decision record, taken while the record is absent or fails `lyx loom validate-discussion` (Step 3's own condition): read `lyx board list`, every entry that touches the task and every `Decision for <slug>:` note for it, and `lyx loom peers` for the runs in flight and the parent's commits since the fork.

- When the entry is plainly outdated in D6's sense and no decision note answers it, the writer asks its parent by `SendMessage`, naming what overtook or contradicts it and asking it to write the answer as a `Decision for <slug>:` board note and then reply by `SendMessage`, since a note alone wakes nothing, and ends its turn to wait.
  On any message from its parent it re-reads the board for the note.
- It writes the answer into its decision record as a Decision citing the parent.
- Once the record is committed, it asks the parent to record the same answer with `lyx loom decision add`, and until the record carries that `Added after Discussion` entry it names the answer in the change description's design decisions as an unrecorded parent correction, as the stencil's scope-correction section does today.
- With no parent, it does not wait, since a held turn end with no parent reaches nobody until `writer_timeout_min`, and builds nothing: it writes the change description, its one output file, as its stop report, naming the outdated entry, what overtook or contradicts it, and the way forward: a `Decision for <slug>:` board note that answers it, then `lyx loom resume`; that ends its turn.
  Darn-Write's `discussion` gate (`attempts: 3`, `contracts/recipes/darn-recipe.yaml`) then fails on the absent record and re-prompts the writer; the stencil tells a re-prompted writer whose stop still holds to end its turn with the stop report unchanged, so the gate exhausts and halts the run, as the recipe header says an exhausted writer gate does, with the gate's findings kept and the stop report in the change description for whoever resumes it.
- An ordinary misalignment it settles itself, as today.

What it can skip or let through: nothing; it only adds a wait, or with no parent a stop before any change.
What bounds it: the parent's reply, which wakes the writer, and otherwise `writer_timeout_min`, the writer's whole-session deadline, shared with the change it writes afterwards (`internal/darnengine/doc.go`, "its wall-clock budget"), since a turn end without every output file is held by shuttle with a hold notice to the orch (`internal/shuttleengine/doc.go`, "A turn end without every output file never ends a run").
A timeout halts the run, since Darn-Write has no `on_stuck`, and nothing respawns the writer automatically; a resume that spawns a writer repeats the check and finds an answer the parent left as a decision note, and one that re-attaches to the waiting writer gives it a fresh deadline, where a parent message makes it re-read the board for the note.

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
The `Parent-Align` row reads the old `Merge-In`-named fixer attempt record (`FixerAttemptsPath`, `Merge-In-fixer-attempts.json`) when its own is absent, and removes the old record wherever it removes its own, so a carried run keeps its spent attempts, `Baseline`, `AlignFrom` and `Halt`.
`internal/reportengine` keeps classifying an archived `Merge-In` row in the merge phase and among the verify rows, as it keeps `retiredPublishRow`.
Reconcile renames a hub's `merge_fix_attempts` and `merge-in` gate key and reports each, and a run profile or `run_settings` holding the old key is migrated at read time with a Warn.

Rationale: since `fc75aef79` the row aligns the task with its parent and only starts with a merge (board entry, Part 3); the fabric verb and git-level merge-ins still only merge.
The loom recipe header's restart rule covers retired rows; a one-to-one rename has the `RenameProducers` precedent (`internal/shedengine/doc.go`, "Row renames").

### Added after Discussion (parent, 2026-10-25): No landing-time approval replaces the alignment agent; D1 stands

Decision: The alignment agent goes whole as D1 says. No landing-time approval is added in its place: the Parent-Align (formerly Merge-In) fixer's check of the task's work against its decision record and what the parent brought in is the landing-time alignment. The entry's 'keep it at pr-gate' was written before fc75aef79 removed the PR-Gate row and is void. Drop the Open risks line that asks this.

Rationale: Parent, 2026-10-25: the PR-Gate point no longer exists and Merge-In's fixer already took over landing-time alignment; the operator holds the alignment agent unneeded now that the Discussion perch's lenses and the periodic board tidy cover its work.

## Constraints

- `PATTERN-refusal-way-forward`: the `decision` escalation's Reason, the hard-cap halt over a held refusal and the arbiter's refusal name the way forward, and a test reaches each.
- `PATTERN-config-layers`: retired and renamed keys are handled only by reconcile and the read-time migration.
- `PATTERN-shed-recipe-registry`: D8's two switches, the Discussion-Bouncer's and the Discussion-Burler-Fix row's, are looked up by key.
- `PATTERN-stencil-ownership`: lens text, rubric and stencils are read at call time; the lenses point at `lyx loom peers` rather than restating its output format.
- `lyx loom peers` lists each peer's row position in the loom recipe; D2 and D11 change no row position.

## Auto-mode assumptions

- D1: PR-Gate premise false; the agent goes whole.
- D4: archived records stay readable by reports.
- D5: `goal-scope` and `board-fit` become procedure lenses and join `review-discussion`.
- D8: a Discussion-Bouncer switch and a Discussion-Burler-Fix switch set together, any severity, replacing the fix row's notice there.
- D11: gate key and fixer mode follow the row name; the merge record keeps its name; runs are carried across.

## Open risks

- `umbrella-daemon` is in flight at Discussion-Write: its entry names parent review among the orch's escalations, and it plans to move the arbiter into shed, which must keep D8's refusal of a `decision` escalation; whichever run lands second takes the other's change at Parent-Align.
- `loom-beam-darns` may run its pieces on darn; D10's check would then run inside a beam and see sibling pieces as peers, which that note's design must settle.
- The scope check now runs after the writer finishes, not before the first perch round, so a round may polish a record whose scope the parent then changes (board entry, Part 1 cost).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d8-hard-cap-refusal-has-no-exit (class design, severity MEDIUM, open in rounds 3)
  - d8-next-fixer-carrier-does-not-exist (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Discussion-Write's only gate is `discussion`; a passing discussion goes to Discussion-Bouncer with no alignment spawn and no parent message.
- `lyx loom review` is no command; `internal/alignment` and `internal/parentreview` are gone.
- A run waiting in a parent-review round passes Discussion-Write on its next gate evaluation under the new binary.
- A hub `loom.yaml` holding a retired alignment or parent-review key loads after reconcile drops it with a report; a profile or `run_settings` holding one loads with a Warn.
- Reports over archived runs still count their parent-review rounds and alignment attempts, and still file archived `Merge-In` rows in the merge phase.
- `review-discussion` resolves to its existing lenses plus `goal-scope` and `board-fit`, each a procedure ending in its list with D6's reads, bounds and parent questions, and each read set at most six calls with its decision-note reads.
- The `discussion` and `twill-discussion` fans keep their members and resolve the rewritten texts; a hub `lenses.yaml` defining `goal-scope` or `board-fit` keeps its own text.
- Discussion-Review round whose accepted review holds a `decision` finding: the fixer never gets it, the fix row posts no notice, and the Bouncer returns Awaiting with cause `decision` after a CONVERGED, a CONTINUE and a CIRCLING verdict alike, and over a BLOCKING `decision` finding without a re-judge, with a brief listing the finding and the way forward.
- The same round with `discussion_auto_accept` true at the soft cap escalates `decision` instead of accepting.
- A re-call over a round escalated after a CONVERGED verdict returns the same Awaiting, with and without a `lyx loom decision add` commit in between.
- `continue` recorded on a `decision` escalation over a CONVERGED round runs another round in the same generation; `accept` settles, carries every `decision` finding, a LOW one included, into `## Open risks`, each naming the `Added after Discussion` entries recorded since the escalation, and goes to Plan-Write.
- A `decision` escalation over a CONVERGED round whose review also holds a BLOCKING `design` finding lists that finding and the refusal's cause in its brief, and on `accept` the round is re-judged before the accept acts, spending one counted bounce unit, with the escalation record and decision file kept, so the round is not re-escalated; a refusal that holds at the budget below the hard cap retires the accept and re-escalates with cause `decision`, its brief naming the held refusal, and a `continue` then recorded runs another round, with an empty focus file, whose fixer gets only that round's own review.
  A BLOCKING `decision` finding alone, whose ledger entry the CONVERGED round holds open, does not refuse the accept.
- A `decision` accept recorded at the hard cap over a judgment the refusal still rejects at the budget blocks the run at Discussion-Bouncer with a budget-exempt halt whose Reason names the refusal's cause and `lyx loom resume`, and does not re-escalate; a resume re-judges the round.
- `continue` recorded on a `decision` escalation over a round whose ledger drops a key of the previous round's ledger re-judges the round before the next round starts.
- A re-entry after a settled `decision` accept re-settles over an unchanged subject or starts a new generation over a changed one, and never degrades.
- A `decision` escalation at or past the hard cap offers accept only, and a `continue` re-escalated on the hard-cap path keeps cause `decision`, is still refused to the arbiter and keeps the round's first `Added after Discussion` snapshot.
- `internal/arbiterstop` refuses a `decision` escalation with `HandBackWayForward`.
- Plan-Review and Code-Review rounds with `decision` findings behave as before: fix-row notice, no escalation.
- The darn writer's stencil carries D10's check, its condition, the ask for a decision note and a reply, the wait, the re-read on a parent message, the later `lyx loom decision add` request, the no-parent stop report and its unchanged turn end on a gate re-prompt; the Discussion writer's stencil carries D9; the rubric carries D7.
- The loom recipe has `Parent-Align` between Webster and Code-Bouncer and as Finalize's reroute target, built by engine `ParentAlign`; no recipe row or engine is named `Merge-In`.
- A status on `Merge-In` resumes on `Parent-Align`; a run past the row at upgrade still gives the Code-Perch its latest fixer report.
- A run halted spent or over-cap on `Merge-In` resumes on `Parent-Align` with its spent attempts, `Baseline`, `AlignFrom` and halt intact.
- A hub `landing.yaml` holding `merge_fix_attempts` and a `gate.yaml` holding `merge-in` are renamed by reconcile with a report, and the renamed values take effect.
- `lyx fabric merge-in` and `fabricengine`'s merge-in names are unchanged.

## Notes for the plan writer

- The escalation cause set D8 extends is checked in `internal/shedadapters/escalation.go` (`EscalationCause.valid`), `internal/shedadapters/circling.go` and `internal/arbiterstop/arbiterstop.go`, and listed in `lyx loom circling --help`.
- The re-entry rule D8 changes is `settledGeneration` in `internal/shedadapters/bouncer.go`, and the replay rule it changes is `standsAsRecorded` there, whose replay settles a recorded round with no refusal today.

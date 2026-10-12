# Support log — merge-review

## Interview

The session ran autonomously; every turn below is an exploration step or a self-made pick, recorded in the question ledger.

1. Read the board entry and the entries it touches: `lyx-halt`, `driver-arbiter`, `config-layers-run-settings`, `alignment-agent`, `orch-split`, `runs-overview`, `orch-any-worktree`.
2. Read the loom recipe: Publish is the only merge-in before PR-Gate, Finalize catches up again before landing, and both recipes' PR-Gate Stuck goes to their rework row only.
3. Read landingshed's and battenshed's package docs: Finalize does the parent-side merge and push; batten's Land row only pulls the prime, deploys and prunes, under the existing land lock, so the parent lock need not cover it.
4. Read the shed loop docs: the detached `--until-stop` loop holds a loop lock, writes a pid-and-start-time record, and exits at every stop, which makes it the natural lock owner.
5. Read the Bouncer docs: auto-accept settles any spent budget, so a budget-1 perch with auto-accept never escalates; the board asks for escalation of an unsettled finding, hence D3.
6. Read shed's producer contract: Stuck routes only to `on_stuck` within a segment, so the moved-parent send-back needs a new edge (D9).
7. Lens review: all six forks reported; duplicates across lenses are merged into one line.
   - scenarios BLOCKING, D9/D10 a parent that already holds the task's work (GitHub merge, earlier landing) read as moved: fixed, moved now excludes a parent containing the task's work and Finalize's already-landed path stays.
   - scenarios + goal-scope MEDIUM, D6 task side counted earlier merged parent changes: fixed, the task side now diffs from the parent tip last merged.
   - scenarios MEDIUM + goal-scope LOW, D6 no writer of "reviewed": fixed, Merge-Bouncer's Done (skip included) records it; a cut-short review records nothing.
   - goal-scope MEDIUM, D5/D13 lock released at every PR-Gate undercut the board: fixed, the loop hands the lock to batten while the orch reviews (D5, D8), bounded by the idle bound.
   - goal-scope MEDIUM, first acceptance criterion inconsistent: fixed with the hand-over and "any release path".
   - goal-scope MEDIUM, perch not optional: fixed, `skip_merge_review` added with a safe zero value.
   - premises + pattern-fit MEDIUM, D5/D7 shedverbs cannot own lock knowledge: fixed, told loop-spec seams filled by the arming modules.
   - premises MEDIUM, D9 two-sided Finalize and records commit: fixed, D9 compares the code side; Finalize's merge-in stays after the check and syncs records.
   - premises LOW, "held" is not a session state: fixed, held turn ends come from shuttle's wait markers.
   - premises LOW, unreadable state in D8: fixed, an unreadable state counts as not waiting.
   - premises LOW, D12 precedent: fixed, named the verify marker in Notes.
   - pattern-fit MEDIUM, D12 SetWaitMark caller: fixed, the mark goes through landingshed's existing callback and D14 updates the entry's caller set.
   - pattern-fit + board-fit MEDIUM, zero values of new keys: fixed, D2 states the zero-value behaviour and D7 reads below-1 as the default via hubreconcile.
   - pattern-fit MEDIUM, reroute history outcome: fixed, a new history-only `reroute` outcome, goto clause unchanged.
   - pattern-fit LOW, told-geometry bound list: fixed in D14.
   - pattern-fit LOW, segment colour: fixed in D2 and Constraints.
   - pattern-fit + board-fit LOW, orch stencil's Finalize and Publish-retry lines: fixed in D13.
   - pattern-fit LOW, batten lock verb refusals: fixed in D11.
   - scenarios MEDIUM, reroute unbounded and lock wait unfair: fixed, a reroute cap of three and a ticket queue.
   - scenarios MEDIUM, D8 could free a fresh loop's lock: fixed, compare-and-release on the owner.
   - scenarios MEDIUM, parent tip undefined: fixed, the upstream tip after a fetch.
   - scenarios MEDIUM, stale wait record: fixed, the wait record carries pid and start time.
   - scenarios LOW, any-agent rule: fixed, no busy agent and one waiting.
   - scenarios LOW, Merge-In takes over clearing parked merges: fixed in D1 and an acceptance criterion.
   - scenarios LOW, pause during lock wait: fixed in D5.
   - scenarios LOW, branch encoding and empty directory: fixed in D4.
   - scenarios LOW, fallback review subject: fixed in D6.
   - board-fit MEDIUM, `config-layers-run-settings` key classification: fixed as an open risk naming the second-to-land task.
   - board-fit MEDIUM, `driver-arbiter` Publish halt row and notice path: fixed as an open risk; D3 names arbiter eligibility.
   - board-fit MEDIUM, D13 against `alignment-agent`: fixed, D13 leaves in-flight alignment to that note.
   - board-fit LOW, `finalize-discrepancy-doc` and `raddle`: fixed in Notes.
   - board-fit LOW, `test-tmp-cleanup`: fixed, Scope Out states why no cleanup hook.
   - board-fit LOW, `lyx-halt` and D7: fixed, D7 states release-all covers a pause.
   - leanness MEDIUM, Auto-mode assumptions repeated Decisions: fixed, cut to pointers.
   - leanness LOW, hand landing stated three times: fixed, kept in Scope Out only.
   - leanness LOW, Scope In restated Decisions: fixed, cut to D references.
   - leanness LOW, Constraints listed unbinding entries: fixed, trimmed.
   - leanness LOW, D12 and Constraints overlap: fixed, the wait-mark line moved to D12 and D14.
   - leanness LOW, D14 rationale: fixed, it now says why the rule is cross-cutting.
   - leanness LOW, Interview turn 7 filler: fixed, this turn now holds the lens review.
   - leanness LOW, Finalize config-file read: fixed by D9 keeping Finalize's merge-in, so the read keeps its anchor.

## Rejected alternatives

- Merge perch after Publish, re-entering Publish to push the fixes: Publish would push an unreviewed merge first, and the perch's on_done back to Publish loops.
- Letting Finalize's catch-up merge bring new parent code: it lands parent commits no review saw, which the task exists to stop; the merge-in stays only behind D9's check.
- An OS file lock (flock) as the parent lock: it dies with the step process that took it, and the lock must span many steps.
- A wall-clock lease with expiry: a hibernated host would expire it, which the board rules out.
- Batten as lock owner throughout: batten lives through every awaiting stop, but the board requires release on a human stop; batten owns the lock only through the orch's PR-Gate review.
- Releasing the lock at the PR-Gate awaiting as at every other stop: every PR-gated run would then lose the lock during the orch's review, which the board's premise for narrowing the PR-Gate fork rules out.
- Removing Finalize's catch-up merge-in: it also syncs the records side the pull request never carried.
- A `merge_review: true` switch: an existing pair's zero value would turn the review off.
- Batten sending the moved-parent run back with goto, as Publish-retry does: a resume plus PR-Gate's reroute keeps the decision under the lock in one place.
- Rerouting through PR-Rework: its Done goes to Plan-Bouncer, which would walk the whole main line again.
- Glyph-level overlap through quarry: a same-package change with disjoint glyphs still interacts through package invariants, so package level is the safer line.
- Changing auto-accept for every perch: unlicensed behaviour change to perches `burler-split-budget` just landed.
- A hand-lock verb for orch landings: a hand landing goes through the run's rows, and a raw push is caught as a moved parent.
- The idle bound in per-worktree `loom.yaml`: a task could weaken a hub-wide fairness rule through its own tree.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the merge-in live? Picked a new Merge-In row ahead of Publish (D1).
- Q2 (auto-pick): Who owns the lock? The run's detached shed loop (D5).
- Q3 (auto-pick): Where does the lock live? A record per parent branch under the board dir's `.lyx` tree (D4), answering the board's first open point.
- Q4 (auto-pick): How do batten's Land row and a hand landing take the lock? Land needs none, since Finalize lands; a hand landing goes through the run's rows (Scope).
- Q5 (auto-pick): The waiting bound and its home: 10 minutes, hub-wide `landing.yaml` `parent_lock_idle_release_min` (D7), answering the board's second open point.
- Q6 (auto-pick): Does a lock wait show as waiting? Yes, in `lyx loom status`, as a wait mark, and as a live wait for batten's quiet notice (D12), answering the board's third open point.
- Q7 (auto-pick): How does the moved parent send the run back? A shed reroute edge from Publish, PR-Gate and Finalize to Merge-In (D9), and batten resumes an awaiting child (D10).
- Q8 (auto-pick): How does "auto-accept on cap" fit "a finding the fix cannot settle escalates"? A Merge-Bouncer-only key escalates over an open BLOCKING finding (D3).
- Q9 (auto-pick): Overlap granularity: file, Go package and PATTERN change (D6).
- Q10 (auto-pick): Does darn get the lock? Yes, with Merge-In and the reroute, without the perch, since an unlocked darn landing would move the parent under a locked loom run.
- Q11 (auto-pick): Where do the force-release verbs live? `lyx batten lock` (D11).
- Q12 (auto-pick): Is the orch's PR-Gate review a "human" stop that releases the lock? No: batten holds it through the review, bounded by the idle bound (D5, D8).
- Q13 (auto-pick): How is the reroute bounded? Three reroutes per row per run, then a blocked halt, and a ticket queue against starvation (D4, D9).
- Q14 (auto-pick): How is the perch made optional? `skip_merge_review`, whose zero value keeps the review on (D2).

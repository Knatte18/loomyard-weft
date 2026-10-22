# Support log — machine-handover

## Interview

### Turn 1 — exploration

Autonomous run: no operator answers, and every question below was settled as an auto-pick.
Read the board entry, its issues #741 and #742 as the body imports them, and the entries that touch it: `orch-any-worktree`, `umbrella-daemon`, `pair-from-parent`, `fabric-add-adopt-local`, `run-process-ownership`, `orch-strand-cycle`, `finalize-dirty-names-paths`, `batten-window-default`, `umbrella-anchor`.
Found no machine identity anywhere in the code, the origin reconcile already landed with its hub and run scopes and callers in hibernate wake, orch start, batten resume, cleanup and loom landing, the hibernate wake already pausing stopped runs through tender, and batten's InnerRun row already reading sleeps and the pause record.

### Turn 2 — scope and identity (auto-picked)

- Q: Where does the machine name come from? Recommended: the hostname, normalised, from a stdlib leaf package. Picked as recommended (D1).
- Q: Where do owners live? Recommended: a task field on Bolt and a hub-level owner record for the prime, both through `boardengine`. Picked (D2).
- Q: Who owns a task never started, and a hub that never recorded an owner? Recommended: the parent's owner, and claim-on-first-act for the prime. Picked (D2).

### Turn 3 — verbs and checks (auto-picked)

- Q: Where does `lyx adopt` sit? Recommended: a top-level module like `lyx cleanup`, prime-only. Picked (D5).
- Q: What does adopt resume? Recommended: every adopted task whose run record is not done, as `--window` starts it. Picked (D5).
- Q: How is a plain suspend, with no `lyx hibernate`, covered? Recommended: InnerRun's owner check after a sleep or a pause hold. Picked (D6).
- Q: Which landing row holds the owner guard? Recommended: Finalize, before the parent lock. Picked (D7).
- Q: How is Open-Window per machine? Recommended: a `.lyx` marker and a one-time run of the row's command at resume. Picked (D8), later replaced at the lens review.

### Turn 4 — lens review

Every one of the seven forks returned a report. Findings, merged across lenses:

- walkthrough, BLOCKING: the reconcile cannot read owners through `boardengine`, since `boardengine` imports `fabricengine`. Fixed: D4 takes an owner reader through `OriginSeams`.
- bound-breaker, BLOCKING: a moving reconcile would replay an owned_elsewhere records branch, publishing the old machine's commits. Fixed: such branches only fast-forward (D4).
- bound-breaker, MEDIUM: the run lock refuses an owner change today. Fixed: D2 exempts `owner_machine` as a decision.
- existing-paths and bound-breaker, MEDIUM: the board replay drops or refuses an owner write. Fixed: `owner_machine` is a replay field where origin wins; the upsert refusal is payload-only; adopt re-reads owners after the push (D2, D5).
- interactions, MEDIUM: a `get` then `upsert` round trip would be refused. Fixed: an unchanged effective owner is a no-op (D2).
- interactions and walkthrough, MEDIUM: the parent fallback silently moves legacy runs at the first prime claim and at `adopt --prime`. Fixed: run-held tasks with no owner are excluded from the fallback and recorded at their next resume (D2, D3).
- interactions, MEDIUM: shed resume verbs bypass the batten check. Fixed: D3 covers `lyx loom start`/`resume` on an existing run; the arbiter and tackler window is in Open risks.
- walkthrough, MEDIUM: adopt does not resume a child halted at Finalize. Fixed: D5 step 5 resumes a child halted at the owner halt only.
- goal-scope, MEDIUM: D7's way forward rewords the board's. Fixed: D7 says so and why.
- goal-scope, MEDIUM: tender status shows nothing for a run with no local pair. Fixed: D9 lists those.
- existing-paths and interactions, MEDIUM/LOW: the landing seam is an unlisted caller, and D7 judged ownership a second time. Fixed: D4 lists the landing seam, and D7 takes the run's verdict from it.
- bound-breaker and interactions, MEDIUM/LOW: D7's "before the parent lock" is wrong, since the lock is already held. Fixed: the check runs before the parent-side merge, and the held lock stays held.
- bound-breaker and goal-scope, MEDIUM/LOW: an unreachable origin is trusted at start and Finalize. Fixed: an owner write needs its push (D3), and Finalize halts (D7).
- bound-breaker and walkthrough, MEDIUM/LOW: InnerRun's record-only pause does not stop the child. Fixed: D6 pauses through a tender pause seam.
- bound-breaker, MEDIUM: the writer bound omits the prime claim. Fixed: D2 names all three callers of the setter.
- bound-breaker, existing-paths and walkthrough, MEDIUM/LOW: the D8 marker survives a reboot, can be swept, and duplicates windows on legacy runs. Fixed: D8 reads whether a reed client is attached instead of keeping a marker.
- board-fit, MEDIUM: `owner` collides with the process owner records of `run-process-ownership` and the strand owner of `orch-any-worktree`. Fixed: the field is `owner_machine`.
- board-fit and bound-breaker, MEDIUM/LOW: adopt's add meets `ErrBranchExists` (`fabric-add-adopt-local`). Fixed: D5 step 2 stops that slug with fabric's message; Scope Out names the note.
- board-fit, MEDIUM: non-prime parents already exist. Fixed: D2 defines a pair parent's owner.
- board-fit, MEDIUM: pause leaks and machine-local Webster transcripts are unstated. Fixed: both are in Open risks.
- board-fit, LOW: D7's run check is something the beam handover must relax. Rejected: without it an old machine lands a run adopted elsewhere, and the beam task owns relaxing it.
- board-fit, LOW: `run-process-ownership` needs an owner read for fabric remove. Rejected: that task designs its own read over this one's setter and seam.
- board-fit, LOW: coupling with `batten-window-default`. Rejected: no conflict.
- goal-scope, LOW: adopting a task whose parent is not owned here. Fixed: the D5 envelope warns.
- walkthrough, LOW: a resume makes no claim commit. Fixed: D3 gives it its own owner commit.
- bound-breaker, LOW: an empty-owner hub, and a hostname change. Fixed: D2 says nothing reads as owned elsewhere before a prime owner exists; Open risks names the rename.
- leanness, MEDIUM: Auto-mode assumptions repeat the decisions. Fixed: one line pointing at the ledger.
- leanness, LOW: Constraints, Open risks, the `session-sync` Out bullet and the first plan-writer note repeat other text. Fixed: cut.
- leanness, LOW: Turns 2 and 3 duplicate the ledger. Rejected: the support log is review-only, and the turns carry the recommendation the ledger does not.

## Rejected alternatives

- A machine name from a config machine-layer key: no current need, and a key every hub must set is an operator step the hostname avoids.
- Storing the owner in batten's run record on the weft: the board design puts it on Bolt, which every reconcile already fetches, and an unstarted task has no run record.
- Unowned tasks startable by any machine: two machines could start the same ready task.
- An implicit takeover when a pair is added from origin: the board design makes takeover an explicit verb.
- An owner check before every push of a live run: a network call per push for a case the premise excludes, where the wake check and the landing guard already bound it.
- Holding every run before sleep on a plain suspend: lyx sees no suspend coming; only `lyx hibernate` does, and it already pauses the hub.
- The landing guard in Merge-In or Publish: those touch only the task branch, and halting there stops work the non-owner can still do.
- Open-Window's done carrying a machine name in the tracked run record: machine-local state belongs under `.lyx`.
- A never-tracked Open-Window marker per machine: it outlives a reboot and a closed window, and a sweep can remove it while the run lives; an attached reed client is the live fact.
- The parent fallback for run-held tasks with no owner: the first prime claim would move a run another machine drives, with no adopt.
- Finalize checking only the parent's owner: an old machine would then land a run adopted elsewhere.
- A bare `owner` field: it collides with the process and strand owners other entries plan.

## Review rounds

### Round 1

- D7's halt message now depends on the failed check, and D5 step 5 resumes an owner-halted child when its parent is adopted here (F1).
- Adopt stops a slug with local commits left by an earlier handover, with `--keep-local` and `--drop-local` as its ways forward.
  Removing the pair through fabric was not named, since `lyx fabric remove` also deletes the pair's branches on origin, and the reconcile's repair steps for a diverged or rewound branch push the old commits before the takeover (F2, self-check).
- D7 no longer says it rewords the board's "let the owner land the run"; Turn 4's goal-scope line records that rewording (F14).
- At the self-check, a run whose Finalize already merged into the local parent keeps its lock under the owner pause, and the both-fail halt names one adopt of both entries (self-check).
- InnerRun releases its child's parent lock on an owned_elsewhere verdict or an owner pause, so the slug pause holds the run (F3, F4).
- InnerRun reads an owner refusal of its own child spawn as an owned_elsewhere verdict, rather than skipping the check (F5).
- The start's check moves before the seed, its claim refuses on a failed owner write, and every owner write is re-read after its push (F6, F11).
- D8's same-machine reopen is stated as intended (F13).

### Round 2

- A run whose Finalize already merged into the local parent finishes its landing push on the old machine, unheld by the owner pause, and D6 bounds that to the one push; `PATTERN-owner-machine` carries the exception (F1).
- Adopt stops an owned_elsewhere slug whose pair reads live, and `--drop-local` refuses a live pair (F2).
- The owner refusal reads as a verdict at InnerRun's one spawn seam, for every spawn on an existing run (F3).
- D7's run-not-owned and both-fail halts name the `--keep-local` form of adopt here (F4).
- D3's offline resume states its bound, and Open risks names it (F5).
- At the self-check, the carve-out lasts only while Finalize runs, its bound names the board mark and the pull request's close, and an adopter-first landing is repaired by resetting the old machine's local parent to origin, never by merging.
  The parent's owner's adopt and the `--keep-local` forms name their own step-1 stop and the resume's code-branch stop, and the offline bound covers either push order (self-check).

### Round 4

- `PATTERN-owner-machine` and the Goal gate a run on no other machine owning it, so an unowned run-held entry and a hub with no prime owner act as D2 and D3 say, and an unowned entry no run holds follows its parent's owner (F1).
- Adopt asks `LivePair` itself for every named owned_elsewhere slug with a local pair, since the reconcile asks it only for a branch it would move (F2, narrowed at the self-check).
- After an adopter-first landing the old machine's pair is not torn down by a reconcile, so its halt note names `lyx fabric remove <slug>` and leaves the owner pause standing over the old batten, and Open risks names the replayed records commit and the left batten record (F3, self-check).
  After an old-machine-first landing, `lyx resume <slug>` lifts the owner pause so its Land runs (self-check).
- D6's Finalize-running exemption reads the child's status at CurrentProducer Finalize, running or failed with a transient mark awaiting its one re-step, together with `parentHoldsTaskWork` (F4, widened at the self-check).
- D7's past-merge skip covers the in-process re-step and the unreachable-origin halt, and a re-entry through InnerRun's `Spawn` seam meets D3's refusal (F5).
- The halt note lists the local parent commits origin lacks and names the reset only when the squash is the only one, else a rebase dropping the squash alone by its SHA, with `git rebase --abort` on a conflict (F6, self-check).

## Question ledger

- Machine name source: hostname, normalised (auto-pick; D1).
- Owner storage: task field and hub owner record on Bolt (auto-pick; D2).
- Unowned task and unowned prime: parent's owner, first-act claim (auto-pick; D2).
- Board upsert and owner: owner kept, a change refused (auto-pick; D2).
- Adopt placement and prime-only rule (auto-pick; D5).
- Adopt resumes in windows (auto-pick; D5).
- Plain suspend coverage through InnerRun (auto-pick; D6).
- Landing guard in Finalize (auto-pick; D7).
- Open-Window per machine: an attached reed client, replacing the `.lyx` marker at the lens review (auto-pick; D8).
- Owner field name `owner_machine` (auto-pick at the lens review; D2).
- Legacy run-held tasks with no owner recorded at their next resume (auto-pick at the lens review; D2, D3).
- Orch role reading of a run owned elsewhere (auto-pick; D9).
- InnerRun's spawns on an existing run: an owner refusal read as an owned_elsewhere verdict (auto-pick at round 1, widened at round 2; D3).
- A run past its parent-side merge when adopted elsewhere: its landing push finishes on the old machine (auto-pick at round 2; D6).
- Local commits from an earlier handover: adopt stops the slug, `--keep-local` keeps them and `--drop-local` archives and drops them (auto-pick at round 1; D5).
- Open-Window on a same-machine resume with no client attached: opens, by intent (auto-pick at round 1; D8).
- The old machine's pair after an adopter-first landing: removed by the operator as its halt note names, with `finalizeEntered` unchanged and the owner pause left standing (auto-pick at round 4; D6).
- A Finalize past its parent-side merge at an unreachable origin: proceeds as today, the skip covering that halt (auto-pick at round 4; D7).

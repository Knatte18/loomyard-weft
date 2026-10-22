---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:129 (D6, the carve-out for a run whose Finalize already merged into the local parent)
    summary: D6's carve-out run keeps the parent lock, which exempts it from the slug owner pause, so it finishes its landing push on a machine that no longer owns it, contrary to D6's holds-as-under-any-pause and the declared PATTERN-owner-machine; its way forward fails once the adopter has landed the run
    fix: Rewrite D6's carve-out sentence to decide the case explicitly. Recommended wording is that a run whose Finalize already merged into the local parent here keeps its lock, is not held by the owner pause because the held lock exempts it from slug entries, and finishes its landing push here, since only the push of a merge made while this machine owned the run remains. State its bound in D6, namely that this lets through one parent push of a landing past its parent-side merge and nothing else. When origin already holds the landing because the adopter landed it first, the push is refused and recorded as a failed push, and the local parent reads diverged at the next hub-scope reconcile, whose stop names the repair. Delete the sentence that says the notice names lyx adopt <slug> here, which lands it. Add the same exception to the PATTERN changes bullet for PATTERN-owner-machine, for example landing a run, bar the push of a Finalize already past its parent-side merge. Add an acceptance criterion that covers both orders, where the old machine pushes first and where the adopter lands first.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:103 (D5 step 1, the stop on local commits)
    summary: D5 step 1 stops an owned_elsewhere slug only on needs_replay, ahead, rewound, rewritten or diverged, but on the machine adopting back, its own owner-paused run still has a live driver, so the reconcile reads that pair live and judges no stop; the owner is then written over the post-handover commits, and the run's next moving reconcile replays them with no keep-or-drop choice
    fix: In D5 step 1, add live to the branch states that stop a named owned_elsewhere slug before its owner write, and say why, namely that a slug-scope owner pause leaves the old driver live and the hub reconcile leaves a live pair unjudged for moves. For a live pair, --keep-local records the owner as for the other states. --drop-local refuses a live pair and names --keep-local as its way forward, since moving a branch under a live driver is unsafe and ending that driver is out of scope (run-process-ownership). Extend the adopt-back acceptance criterion to the case where the old run's driver is live under its owner pause.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:62 (D3, the InnerRun spawn bullet)
    summary: D3 rests on a false premise. The bootstrap spawn does not start only a fresh child. InnerRun's one Spawn seam runs lyx loom start --no-attach on existing runs in several places, and those refuse under D3's check. Outside the binary re-step, InnerRun treats such a refusal as a hard error and fails the row, where D6 would pause it.
    fix: Replace D3's bootstrap sentence with a rule over the seam rather than one site. Every InnerRun spawn of lyx loom start --no-attach on a run whose status file exists reads the owner kind in the refusal's envelope as an owned_elsewhere verdict and answers it as D6 does, never as a failed spawn. This covers the re-step, the resume of an awaiting child after a decision or a moved parent, the publish retry's resume and the respawn of an unconfirmed running child. Only the spawn with no status file is the fresh bootstrap that batten's start already judged. Widen the acceptance criterion from the re-step spawn to any InnerRun spawn on an existing run.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:147 (D7 halt messages for the run-not-owned and both-fail cases) and :221 (acceptance criterion)
    summary: D7 says lyx adopt <slug> here, or lyx adopt --prime <slug>, resumes the halted run. The halting machine of a run owned elsewhere carries records commits the adopter's origin lacks, the halt's own included, so D5 step 1 stops that slug before its owner write, and the named call resumes nothing. The acceptance criterion's claim that either lands the run is unmet.
    fix: In D7's run-not-owned and both-fail bullets, say that the named adopt may first stop on the slug's local commits and that its envelope then names --keep-local and --drop-local. Say that lyx adopt <slug> --keep-local, or lyx adopt --prime <slug> --keep-local, records the owner and resumes the halted run through D5 step 5. Change the halt message text to name the --keep-local form directly. Rewrite the acceptance criterion at line 221 to name the --keep-local form for the here-side way forward. Keep this consistent with F2's live-pair stop, since after F2 a halted child whose driver strand is still live also stops at step 1.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:70 (D3, the unreachable-origin fallback)
    summary: With origin unreachable, D3's resume trusts the local Bolt, which is a downgraded owner guard, and states no bound. A run adopted elsewhere after this machine's last fetch is resumed and driven here, D6 does not check it because no sleep or pause hold ended, and only D7's Finalize halt stops it.
    fix: After the fallback sentence in D3, add its bound. A resume with origin unreachable can drive a run adopted elsewhere since this machine's last fetch. The next D6 check after a sleep or a pause hold catches it once origin answers, and so does the landing reconcile, which halts Finalize (D7). Its task-branch pushes meanwhile are refused once the adopter has moved the branch, and are recorded as failed pushes. Add one Open risks bullet saying the same.
---

### [BLOCKING] D6's carve-out lands a run on a machine that no longer owns it, and its way forward can fail

**Location:** `_lyx/discussion/decision-record.md:129`, D6.

**Issue:** D6 says the release of the parent lock is what lets the slug pause hold a lock-holding run: "A held parent lock exempts the child from slug entries, so only the release lets the slug pause hold it."
That exemption is real: `internal/pauserecord/doc.go` says "A caller whose run holds a parent lock is exempt from slug entries, so only a hub entry holds it", and `pauserecord/hold.go`'s `SlugEntryExempt` serves it.
The carve-out then says a run whose Finalize already merged into the local parent keeps its lock.
So that run is not held by the owner pause D6 adds next ("holds as under any pause" is false for it).
Its Go driver goes on through Finalize's remaining steps, marks the board done and pushes the parent (`internal/landingshed/doc.go`: "After the parent-side merge and before the push, Finalize marks the board task done").
That is a landing on a machine that, after the wake, does not own the run.
It contradicts the declared `PATTERN-owner-machine` text, "Starting, resuming or landing a run, or acting on an entry after a wake: only on the machine that owns it", and D7's title.
The same sentence's way forward also fails in a reachable case.
If the adopter ran `lyx adopt --all` it owns both the prime and the run, so its Finalize lands from origin's task branch.
On the old machine, `lyx adopt <slug>` is then refused because D5 refuses a done slug, and the old machine is left with an unpushed squash merge in its local parent and a lock record, with no way forward named.
The design has to pick one behaviour for this run and bound it; the fix picks letting the push finish, since only the push of a merge made under ownership remains.
Round 1's fixer-report added this carve-out at its self-check, and for this one run it undoes round 1's F4 fix, which made the lock release the thing that lets the slug pause hold the run.

### [MEDIUM] Adopting back with the old run's driver still live skips the local-commit stop

**Location:** `_lyx/discussion/decision-record.md:103`, D5 step 1.

**Issue:** Focus item 2 asks what happens when a machine adopts back a slug on which it made records or code commits after an earlier handover.
D5 step 1 stops such a slug only for the branch states needs_replay, ahead, rewound, rewritten or diverged.
On the machine adopting back, the run was paused by D6's slug-scope owner pause, and a tender pause interrupts turns but kills no process (`internal/tenderengine/doc.go`: "it kills no pane, strand or process").
`internal/pairteardown/doc.go` defines `LivePair` as live for a live driver strand, run lock or loop lock, and reads not live only under a *hub* pause entry.
`internal/fabricengine/doc.go` says a pair `LivePair` reports live "is judged and left where it was, reading live once origin moved past it".
So the pair reads `live`, which is not in step 1's list.
The slug goes on, step 3 records the owner, step 4 lifts the owner pause and the old batten resumes.
The run's own moving reconcile at Merge-In or Finalize never asks about its own pair's liveness, and it replays the records branch (`needs_replay` "a moving reconcile replaces with the result of its replay").
The post-handover records are therefore published with no choice between keep and drop, which step 1 exists to force.
A diverged code branch stops there with the reconcile's way forward, but only at the landing rows.

### [MEDIUM] D3 assumes only the re-step spawn meets the owner check on an existing run

**Location:** `_lyx/discussion/decision-record.md:62`, D3.

**Issue:** D3 says "The bootstrap spawn starts a fresh child under a run batten already judged ... so neither meets the check", and gives the owned_elsewhere reading only to "InnerRun's re-step spawn".
Batten has one `Spawn` seam, `exec.Command(exe, "loom", "start", "--no-attach")` in `internal/battencli/wire.go`, and `internal/battenshed` calls it in four places.
The call at `innerrun.go:407` runs when there is no status file and also when a running child's spawn is unconfirmed, which is an existing run.
The call at `innerrun.go:964` resumes an awaiting child after a decision, or after a moved parent at PR-Gate.
`publishretry.go:278` resumes a blocked child through its bootstrap, and `innerrun_binaryrestep.go:127` is the re-step.
All of them except the no-status case run `lyx loom start` on an existing run, which D3's second bullet now gates with a judge-only owner reconcile.
`resumeAwaitingChild` treats any refusal that is neither `ErrChildNotParked` nor `ErrChildDriverBusy` as `hardEnd`, and the `mustSpawn` path returns the spawn error, so an owner refusal there fails the InnerRun row.
That is the "failed spawn" outcome D3 rules out for the re-step.
D6's bound says the child can act for up to one check interval after a wake before the owner check runs, so an awaiting child decided in that window takes this path.
The fix is one rule over the seam, not a list of call sites, so this is a missing rule rather than an enumeration gap.
Round 1's F5 offered the refusal-as-verdict reading for batten's child spawns, and its fix limited that reading to the re-step, because it assumed the bootstrap spawn always starts a fresh child; the code above shows that assumption is false.

### [MEDIUM] D7 says a single adopt resumes a halted run, but D5 step 1 stops it

**Location:** `_lyx/discussion/decision-record.md:147`, D7, and the acceptance criterion at `:221`.

**Issue:** Focus item 1 asks whether every way forward in the halt message leads to a landed run.
When Finalize finds the run owned elsewhere, another machine adopted it after this one last drove it.
Two sources give the halting machine records the adopter's origin lacks.
One is its own halt, which commits the status and writes the halt note.
The other is every records commit it made between the adoption and the halt.
The adopter's own run, resumed by its adopt (D5 step 5), writes to the same records branch, so those local commits read ahead or diverged.
D5 step 1 then stops the slug before its owner write and names `--keep-local` and `--drop-local`.
So D7's "`lyx adopt <slug>` here, which resumes the halted run" and "`lyx adopt --prime <slug>` ..., whose step 5 resumes the halted run" each take a second call.
The acceptance criterion "either lands the run" is not met by the call it names.
For the parent-not-owned case the way forward holds: the run is owned here, and adopting the parent touches only the parent's branches.

### [LOW] D3's offline fallback gives no bound

**Location:** `_lyx/discussion/decision-record.md:70`, D3.

**Issue:** "With origin unreachable every other check reads the local Bolt", and a resume of a task the local Bolt shows as owned here "keeps today's behaviour, a Warn and the resume".
This downgrades the owner guard to a possibly stale local read, and per the rubric's attack-surface rule the Decision must state what that lets through and what bounds it.
D6's retry ("retried at each later check until origin answers") applies only once a sleep or a pause-hold check has run, so a resume made offline is not re-judged until one of those happens or Finalize's reconcile halts (D7).
The Open risks name the offline prime claim, but not this case.

## Rejected

- D3 bullets 1 and 3: the start's owner check can come before the seed.
  `internal/battencli/cli.go` puts the board claim at the head of the Worktree-Create row, after batten's seed, so a read in `lyx batten run` before the seed precedes it.
  The window between the read and the claim is closed by the post-push re-read.
  A failed claim refuses the row, and the resume of that stopped run either re-claims or is refused offline, so it ends in a stop with a way forward and is not a defect.
- D7's both-fail branch naming `lyx adopt <slug>` on the parent's owner: with two machines that owner already holds the run, so the call is a harmless no-op beside the run its earlier adopt resumed. Not a defect.
- D6's release through InnerRun's existing `ParentLock.Release` seam: `innerrun_parentlock.go` already releases records owned by the child's process, so the seam D6 names exists. Not a defect.
- D2's run-lock exemption for `owner_machine`: `internal/boardengine/runlock.go` already leaves `RunSettings` and `Profile` out of the comparison, and only the owner setter writes the field, so the exemption is bounded. Not a defect.

# Decision record — machine-handover

## Goal

Two machines of the operator work on one hub, never at once, and one takes over what the other was doing with one explicit verb and no tangle.
Every task and the prime record an owner machine on Bolt; a machine starts, resumes and lands only what no other machine owns, bar finishing a landing already past its parent-side merge, `lyx adopt` moves ownership here, and a machine that finds a run it drove now owned elsewhere pauses it instead of driving it on.
This closes issues #741 (no ownership, unchecked resume, a missing pair read as landed) and #742 (Open-Window counted done across machines).

## Scope

In:

- This machine's name, and an owner machine per task and for the prime, recorded on Bolt and shown by `lyx board`, `lyx batten status` and `lyx tender status`.
- The owner check on `lyx batten run` (start and resume) and on the shed resume verbs, and the refusal on a machine that does not own the entry.
- `lyx adopt <slug>...`, `lyx adopt --prime` and `lyx adopt --all`, with `--keep-local` and `--drop-local`.
- The owner outcome in `fabricengine.ReconcileOrigin`, and what each of its callers does with it.
- The owner check of a live batten run after a sleep or a pause hold ends.
- The landing guard: a run lands only on the machine that owns it and its parent.
- Open-Window counted per machine.
- The orch role's reading of a run owned elsewhere, the module docs, `docs/overview.md` and the PATTERN entry below.

Out:

- Beams and their owners, and a parent's owner landing another machine's run through a ready-to-land handover on Bolt: recorded in note `orch-any-worktree`, not built here.
- Stopping a driver that is live on another machine at the moment of adoption: the premise is that the machines never run at once.
- Ending a run's processes on the machine it left, and the pause's own leaks: tasks `run-process-ownership` and `tender-pause-holds-reprompt`.
- `lyx fabric add` adopting a local warp branch: note `fabric-add-adopt-local`.

## Decisions

### D1. This machine's name comes from one leaf package

A new leaf package (working name `internal/machinename`, stdlib only) returns this machine's name: the hostname, cut at its first dot and lowercased, so `Neve` reads as `neve`.
An empty or unreadable hostname is an error naming the cause, and no owner is written or compared with an empty name.
Every caller takes the name through a seam, so a test hands in two names for two clones.
Rationale: the operator's machines already have distinct hostnames, and no config layer is needed for a fact the OS already holds.

### D2. Owners live on Bolt, written only through `boardengine`'s owner setter

- A task carries an `owner_machine` field in its board JSON; a note carries none.
  The prime's owner lives in one hub-level owner record on Bolt.
  Both are written only by `boardengine`'s owner setter, whose production callers are batten's claim and resume (D3), the first-act prime claim below and `lyx adopt` (D5).
- The run lock exempts `owner_machine` from its scope comparison, as it exempts `run_settings`, so the owner of a run-held entry can change.
- `lyx board upsert` and `merge` accept an `owner_machine` equal to the entry's effective owner as a no-op and refuse any other value, naming `lyx adopt` as the way forward; the refusal sits in the payload check only, so a replayed owner write is never refused by it.
- The board replay carries `owner_machine` as a replay field, and the hub owner record has the same rule: when both sides changed it, origin's value stands and the replay reports the local change lost.
- The parent of an entry is the prime, or the pair whose branch its origin record names as parent; the parent's owner is the prime's owner or that pair's task's owner.
- A task no run holds and with no owner recorded is owned by its parent's owner.
  A run-held task with no owner recorded, one an older binary started, has no owner until the next machine that runs its batten resume records itself (D3); until then no check reads it as owned elsewhere.
- A hub with no prime owner recorded gets one at the first `lyx orch start`, `lyx batten run` or `lyx adopt` on any machine, which records itself and pushes Bolt; a machine whose write the replay reports lost reads the other's claim.
  Until a prime owner exists, no entry reads as owned elsewhere.
Rationale: Bolt is the one shared record every machine fetches in the origin reconcile, and the parent fallback keeps a hub that never recorded an owner working on its first machine with no operator step, without moving a run some other machine already drives.

### D3. `lyx batten run` and the shed resume verbs act only on what this machine owns

- Before a start, batten pulls Bolt and reads the entry's effective owner before it writes a seed or claims anything.
  A slug with no board entry has no owner, and its start writes none, as today's claim skips an absent entry.
  Before a resume, it takes the owner verdict from its run-scope origin reconcile (D4).
- The shed resume verbs on an existing run (`lyx loom start`, `lyx loom resume` and their darn counterparts) take the same verdict from a judge-only run-scope reconcile.
- When another machine owns the entry, the verb refuses before it creates, seeds, spawns or resumes anything, naming the owner and `lyx adopt <slug>` from this machine's prime as the way forward.
- Every InnerRun spawn of `lyx loom start --no-attach` on a run whose status file exists passes the same check: the re-step, the resume of an awaiting child after a decision or a moved parent, the publish retry's resume and the respawn of an unconfirmed running child all go through InnerRun's one `Spawn` seam.
  The refusal carries an owner kind in its envelope, which InnerRun reads at that seam as an owned_elsewhere verdict and answers as D6 does, never as a failed spawn.
  Only the spawn with no status file is the fresh bootstrap, under a run batten's start already judged, and a revive relaunches the driver strands through reed resume with no shed verb, so neither meets the check; D6's checks bound both.
- A start records this machine as the task's owner in the claim's Bolt commit.
  The claim, which today only warns on a board failure, now refuses the Worktree-Create row when that owner write or its push fails, and the stopped run resumes with the same verb.
  Batten's resume of a task with no owner recorded writes the owner in its own Bolt commit and push before it resumes; a refused push refuses the resume.
  After either push the start or resume re-reads the owner from Bolt, and an owner that does not read as this machine, because the replay kept origin's value, refuses naming that owner and `lyx adopt <slug>`, as D5 step 3 does.
  The shed resume verbs on a run-held task with no owner recorded proceed and write no owner.
- An owner write needs its push: a start or an owner-writing batten resume with origin unreachable is refused, naming a re-run once origin answers.
  With origin unreachable every other check reads the local Bolt.
  Batten's resume on a task this machine owns there, or on a run-held task with no owner recorded, keeps today's behaviour, a Warn and the resume, and the shed resume verbs, which run no reconcile today, do the same.
  A task the local Bolt shows as owned elsewhere is refused, naming the owner and a re-run once origin answers.
  Bound: a resume with origin unreachable can drive a run adopted elsewhere since this machine's last fetch.
  The next D6 check after a sleep or a pause hold catches it once origin answers, but none may come before Finalize, whose landing reconcile is certain to catch it and halts it (D7).
  Meanwhile both machines drive the run, and either side's task-branch push can be the one refused, the adopter's included when this machine pushed first; each refusal is recorded as a failed push.
Rationale: issue #741's resume ran with no check; these are the gates every start and resume passes.

### D4. The origin reconcile reports an owner outcome

- `ReconcileOrigin` takes an owner reader through `OriginSeams`, which the caller builds over `boardengine` as it builds the other seams, since `boardengine` imports `fabricengine`.
  The reader is handed origin's fetched Bolt tip, the same tip the reconcile reads the ended set from, so a judge-only scope reads owners too and moves nothing.
  A nil reader reads every entry as owned here.
- An entry whose effective owner is another machine gets a new entry state, `owned_elsewhere`, carrying the owner's name and the way forward `lyx adopt <slug>`.
  Precedence: ended, then owned_elsewhere, then stopped, then active.
  The report's `Summary` names each owned_elsewhere entry with its owner.
- An owned_elsewhere entry's branches only fast-forward: a branch judged needs_replay, ahead, rewound, rewritten or diverged stays where it is and is reported, so no commit the old machine made after the handover is replayed onto the new owner's tip.
  A live pair stays where it is, as today.
- Each caller acts on the outcome:
  - the hibernate wake gives each owned_elsewhere entry with a local pair a tender pair pause, as it does a stopped run, and its notice names the owner and `lyx adopt <slug>`; the pause holds a run whose child holds the parent lock only once its batten releases that lock (D6);
  - `lyx orch start` gives each owned_elsewhere entry with a local pair a tender pair pause, reports it, and still never refuses; its pause holds a lock-holding run the same way;
  - batten's resume and the shed resume verbs refuse as D3 says;
  - the landing rows' run-scope reconcile seam passes the verdict to Finalize (D7), and Merge-In, Publish, PR-Align and PR-Gate proceed on it;
  - `lyx cleanup` only reports it, and tears no such pair down.
- The pause entries these callers and D6 add carry one fixed `Caller` label marking them as owner pauses, so `lyx adopt` lifts exactly those (D5).
  A caller that finds a live entry already on the slug's scope leaves that entry standing and adds none, as tender adds no second live entry per scope.
Rationale: the board entry assigns this outcome to this task, and the reconcile is the one judge every wake and start already runs (`PATTERN-origin-reconcile`).

### D5. `lyx adopt` takes over exactly the entries it names

- A new top-level module (working name `internal/adoptcli`), a cobra root child with no engine of its own, calling each owning module's function in-process, as `lyx cleanup` does.
  It runs from the hub's prime only, by batten's prime refusal.
- Arguments: one or more task slugs, `--prime` alone or with slugs, or `--all`, which names the prime and every task that is not done and has an owner or a run status.
  A slug that is unknown, a note or done is refused before anything runs.
  `--prime` writes the prime's owner only, so every task with an owner keeps it, and only the unowned tasks no run holds follow the prime here.
  `--keep-local` and `--drop-local` settle step 1's stop on local commits or a live pair for each named slug, and change nothing for a slug step 1 does not stop.
- Steps, in order:
  1. A moving hub-scope origin reconcile, which brings warp, the prime's records branch, Bolt and every local pair up to date.
     An unreachable origin, a reconcile error, or a stop on Bolt or the prime's records branch refuses the verb with the report and its way forward, and nothing below runs.
     A named slug the reconcile reads as owned_elsewhere, whose records or code branch it judged needs_replay, ahead, rewound, rewritten or diverged and so left in place (D4), or whose pair adopt's own `LivePair` call reads live, stops before its owner write, and the other slugs go on.
     The reconcile asks `LivePair` only for a branch in state fast_forward or needs_replay, so its report says nothing of the liveness of a pair whose branches it judged ahead, diverged, rewound or rewritten;
     adopt therefore calls `LivePair` of the `OriginSeams` it built itself, after the reconcile, for every named slug the reconcile reads as owned_elsewhere and that has a local pair, so a run this machine owns and drives is never stopped, and the stop, the `--drop-local` refusal and the `--keep-local` record all read that answer.
     A live pair stops too because a slug-scope owner pause leaves the old driver's strands alive, though interrupted, and the hub reconcile leaves a live pair unjudged for moves, so its post-handover commits would otherwise be replayed by the run's next moving reconcile with no choice.
     The stock seams cannot see such a pair: `pairteardown.HubSeamOptions` fills `HubPaused` from `pauserecord.Record.Covering`, which matches a slug-scope entry as well as a hub entry, and `OriginSeams.LivePair` reads a pair `HubPaused` covers as not live.
     So adopt builds its reconcile's seams with a `HubPaused` that covers hub entries only, and a pair whose strands are alive under any slug-scope pause, the owner pause included, reads live; under a hub pause LivePair reads not live as today.
     Its envelope entry names each such branch and its state, and two ways forward, neither of which pushes before the owner write:
     `lyx adopt <slug> --keep-local`, which records the owner over those local commits, so the resume's moving reconcile replays or stops on them as it does for any run owned here, and records it the same way for a live pair;
     or `lyx adopt <slug> --drop-local`, which keeps each such branch's local tip under a local `archive/<slug>/<tip>` tag, moves the branch to origin's tip, and goes on, stopping as the reconcile's dirty state does when a tracked change sits in its worktree.
     `--drop-local` refuses a live pair and names `--keep-local` as its way forward, since moving a branch under a live driver is unsafe and ending that driver is out of scope (`run-process-ownership`).
  2. Each named task with a pair on origin and none here gets its pair through fabric's add from origin; a refusal of that add, such as a local warp branch with no worktree, stops that slug with fabric's message and way forward, and the other slugs go on.
  3. One Bolt commit records this machine as the owner of every named entry that steps 1 and 2 did not stop, the prime included when named, and pushes.
     A refused push refuses the verb, and no run resumes.
     After the push adopt re-reads the owners from Bolt, and a slug whose owner does not read as this machine, because the replay kept origin's value, stops with the owner it reads and starts no run.
  4. Tender lifts each live entry carrying D4's owner label whose scope is an adopted slug, by that entry's scope, caller and paused time; any other pause entry, a hub pause included, stays.
     A slug still covered by another entry gets an envelope line naming the standing pause and its lift: `lyx resume <slug>` for an entry on the slug's own scope, and `lyx resume` for a hub entry, which tender refuses to lift by slug.
  5. Each adopted task with a batten run record that is not done is resumed as `lyx batten run <slug> --window` resumes it, and the run's own driver lock refuses a second driver here.
     A named task with no run record gets its owner recorded and no run.
     A child halted at D7's owner halt is resumed as `lyx loom resume` resumes it, both when its own slug is adopted and when `--prime` or a parent slug adopts its parent here while this machine owns its run;
     a child halted for any other reason stays halted for the orch.
- The envelope carries the reconcile report, the `mutations` and `partial` pair of a mutating verb, and per slug what happened: a stop on local commits or a live pair with its branches, pair added, owner recorded, pause lifted or standing, run resumed or none for a task with no run record, the error, and a warning when this machine does not own the slug's parent, since that run halts at Finalize here (D7).
- Bound: adopt writes owners without the old owner's help and does not stop a driver live elsewhere; that machine stops at its next owner check (D3, D4, D6, D7).
Rationale: the board design makes takeover one explicit verb, and steps 1 to 3 run before any resume so no run is driven here before origin records this machine as its owner.

### D6. A live run re-checks its owner after a sleep or a pause hold

- Batten's InnerRun row, which already reads sleeps through its Awake seam and the pause record each check, runs a judge-only run-scope origin reconcile when a sleep of at least a minute ends and when a pause hold over the slug ends.
- An owned_elsewhere verdict, or an owner refusal of one of InnerRun's own spawns on an existing run (D3), first releases the parent lock its child's run holds, through InnerRun's existing parent lock Release seam, which releases only a record naming its own child's run.
  A held parent lock exempts the child from slug entries, so only the release lets the slug pause hold it.
  This is the one release batten's lock step makes under a pause, which otherwise keeps a covered holder's lock.
  A run whose Finalize is running and already merged into the local parent here keeps its lock.
  Both signals must hold: the child's inner shed status, which InnerRun already reads for its halt report, reads CurrentProducer Finalize with State running, or with State failed and a non-empty `Transient`, the mark of a failure the shed driver re-steps once, while the run's settle records hold no restep for that stop;
  and the local parent holds the task's work by the same `parentHoldsTaskWork` test that exempts it from Finalize's owner checks (D7).
  The failed arm keeps the lock between a transient parent-push failure and its re-step, since a pause covering the run holds the driver before it re-steps.
  A read failure of that test, while the status reads either arm, keeps the lock too until the next check, since that merge sits unpushed in the parent and would read as a moved parent to the next holder.
  The held lock exempts it from slug entries, so the owner pause does not hold it, and it finishes its Finalize here: only the landing of a merge made while this machine owned the run remains.
  That lets through, past the parent-side merge, Finalize's board-done mark, its parent push and, after a pushed landing, its close of the pull request, and nothing else.
  The exemption lasts only while Finalize runs or awaits that one re-step: once the child's status reads any other state or producer, a stop at Finalize the driver does not re-step or Done included, InnerRun's next pause read releases the lock as for any other run, and the owner pause holds it.
  When the old machine pushes first, the landing reaches origin.
  The adopter's reconcile reads the entry ended, which precedes owned_elsewhere (D4), once the board mark reached origin; the mark is best-effort, and without it the adopter's Finalize takes its already-landed path.
  When the adopter landed the run first, the old machine's parent push is refused, recorded as a failed push, and Finalize stops, so the lock is released as above.
  The local parent then holds a squash of a task origin already landed, so its repair is to drop that commit, and never a merge, which would land the task twice.
  The halt note lists the local parent commits origin's parent lacks and names which of them is the task's squash, by its SHA.
  When the task's squash is the only one, it names moving the local parent to origin's tip, `git reset --keep origin/<parent-branch>` in the parent's worktree;
  when the list holds another commit, such as a landing not yet pushed, it names dropping the squash alone, `git rebase --onto <squash>~1 <squash> <parent-branch>` in the parent's worktree with the squash's SHA filled in, since the reset would drop that landing too, and on a rebase conflict `git rebase --abort` and a drop of the squash by hand.
  The parent's records branch needs no repair: the weft is never a merge participant, so the merge made no records commit, and the records commit Finalize's step 2b made is replayed by the next moving reconcile as any local records commit is, or stops there with its own way forward.
  The halt note names either repair only when origin's parent tip, fetched at the stop, holds the task's work, the test Finalize's already-landed path applies;
  when it does not, or that fetch fails, the note keeps today's reason and way forward, the failed push's own class or a push by hand, since dropping the squash there drops a landing nobody else made.
  When the old machine pushed first, the owner pause still covers its slug once Finalize ends, so its batten's Land waits under it;
  once origin's board marks the slug done, `lyx resume <slug>` lifts that pause, since `lyx adopt` refuses a done slug, and the run goes on to Land and tears its pair down as today, a Land the pause halted resuming by its halt's own way forward.
  When the adopter landed first, the old machine's reconcile never reads the slug ended: the Finalize history row the stop leaves makes `finalizeEntered` true, and after the repair origin's parent still holds the task's work, so `parentHoldsTaskWork` stays true, and `withoutFinalizeEntered` drops the slug from the ended set.
  No reconcile tears that pair down, and the step 2b records commit is replayed by the next moving reconcile, not set aside as an ended run's.
  So the halt note names, after the repair, `lyx fabric remove <slug>` to remove the pair, and leaves the owner pause standing, since a lift would free the old machine's batten, whose run record is not done, to act on the removed pair;
  that batten and its record stay until `run-process-ownership` ends a run's processes on the machine it left (Open risks).
  The row then pauses the pair through a new tender pause seam with D4's owner label, so tender interrupts the child's running turns; it sends one notice naming the owner and `lyx adopt <slug>`, and holds as under any pause, bar that one run, which the pause does not hold while its Finalize runs.
  A released run takes the lock again at its next lock-taking row if it is ever adopted back.
- When InnerRun's pause read finds an entry with D4's owner label over its slug, as the wake and `lyx orch start` add, it releases its child's run parent lock the same way, with no reconcile of its own.
- An unreachable origin is retried at each later check until origin answers, and the run is not held meanwhile.
Bound: the child can act for up to one InnerRun check interval after a wake before the check runs, and for as long as origin stays unreachable.
A landing row past its lock take when the lock is released finishes its step without the lock if a pause over it is lifted, and a call already past the network gate completes.
Rationale: a plain suspend, unlike `lyx hibernate`, pauses nothing before sleep, and issue #741's Neve drove the runs on the moment it woke; `lyx resume <slug>` on an owned-elsewhere run would otherwise drive it again unchecked.

### D7. A run lands only on the machine that owns it and its parent

- Finalize, the landing row that merges into the parent's local worktree and pushes it, checks before the parent-side merge that this machine owns the run, by the landing seam's verdict (D4), and owns the run's parent (D2), read from Bolt as that reconcile left it.
  Both checks are skipped for a Finalize this machine's own landing has reached past the parent-side merge, such as the shed driver's in-process re-step after a transient parent-push failure, whose merge reads already up to date and goes straight to the push.
  A re-entry through InnerRun's `Spawn` seam, such as its binary re-step or the respawn of an unconfirmed running child, meets D3's owner refusal first, which has no past-merge exemption, so InnerRun pauses the pair as D6 says and the local parent keeps the unpushed merge; once the adopter lands, that parent needs the repair D6 names for an adopter-first landing, which no halt note names here (Open risks).
  The test is whether the local parent branch already holds the task's work, `parentHoldsTaskWork` in `internal/fabricengine`, the narrowest part of the reconcile's `finalizeEntered` test, exported for Finalize and InnerRun (D6).
  `finalizeEntered` as a whole is not the test, since a Finalize that halted or rerouted before its merge leaves a Finalize history row it reads as reached.
  A read failure of that test halts Finalize through the halt path, naming a re-run, so a doubt never skips the owner checks.
- A failed check halts the run at Finalize through the normal halt path, so its halt note is written (`PATTERN-friction-capture`).
  The parent lock the run already holds is local to this machine and follows batten's lock step as today: a covered holder keeps it, and an uncovered halted child's lock is released once its agents have waited for the idle bound.
  Once the run is adopted elsewhere, its batten's next owner check releases a lock its pause would keep (D6).
- The message depends on the check that failed, and names the owner it read:
  - when this machine does not own the parent, it names the parent's owner and two ways forward: adopting the parent here, `lyx adopt --prime` for the prime or `lyx adopt <parent-slug>` for a pair parent, which resumes the halted run (D5 step 5);
    or `lyx adopt <slug>` on the parent's owner, which moves the run there and resumes it to land from its branch on origin;
  - when this machine does not own the run, it names the run's owner and `lyx adopt <slug> --keep-local` here, which records the owner over the slug's local commits and resumes the halted run (D5 step 5);
  - when both fail, it names both owners and two ways forward: `lyx adopt <slug>` on the parent's owner, which moves the run there;
    or adopting both here in one call, `lyx adopt --prime <slug> --keep-local` or `lyx adopt <parent-slug> <slug> --keep-local`, whose step 5 resumes the halted run; the pair-parent form keeps the parent slug's local commits too, since `--keep-local` settles each named slug.
  In the last two cases the run owned elsewhere carries records commits the adopter's origin lacks, the halt's own included, or a driver strand still live, so a plain `lyx adopt <slug>` here stops at D5 step 1 and its envelope names `--keep-local` and `--drop-local`; the halt message names the `--keep-local` form directly.
  The resume after `--keep-local` replays those records commits, and a code branch both machines moved stops at that resume's moving reconcile with its way forward, as for any run owned here.
  `lyx adopt <slug>` on the parent's owner stops the same way at D5 step 1 when that machine drove the run before and holds local commits or a live pair for it; the halt message says so, and that adopt's envelope names `--keep-local` and `--drop-local`.
  The halting machine's run is then owned elsewhere and paused at its next owner check.
- An unreachable origin at the landing's reconcile halts Finalize the same way, naming a re-run once origin answers.
  The past-merge skip covers this halt too: a Finalize past its parent-side merge proceeds through an unreachable origin as it does today, and its parent push meets the unreachable origin as a failed push, whose one transient re-step returns here before the tackler takes the stop.
- The check sits in the landing producer, so loom and darn runs and an alignment agent's approve that lands with no orch turn all pass it.
Bound: Merge-In, Publish, PR-Align and PR-Gate run on a machine that does not own the parent, since they push only the task branch and merge into no parent.
A Finalize past its parent-side merge when its entry becomes owned elsewhere finishes on the old machine, as D6's carve-out bounds, unless it re-enters through InnerRun's `Spawn` seam, which D3 refuses.
Rationale: the parent-side merge needs the parent's local worktree, which only the parent's owner keeps current.

### D8. Open-Window opens on a machine with no window attached

- A resume whose Open-Window row is done in the run record runs the row's command once before Run-Shed when the row's switch is on (`open_window`, or `--window`) and reed reports no client attached to the pair's reed session on this machine.
- This covers a same-machine resume by intent: a run the operator detached from, or one the orch resumed, opens a window again at its next resume when the switch is on.
  An operator who wants no window on a task turns `open_window` off for it through its run settings.
- A first run of the row is unchanged.
Rationale: issue #742; the run record travels with the weft, but a terminal window belongs to the machine and the boot that opened it, and an attached client is the live fact a marker would only approximate.

### D9. Every status view shows the owner

- `lyx board list` and `get` carry `owner_machine` on each task, the effective owner when none is set, and the README's Running rows name it.
- `lyx batten status <slug>` carries `owner_machine` and whether this machine owns it.
- `lyx tender status` names each local pair's owner, and lists each run-held entry owned by another machine with no local pair as owned by that machine.
- The orch role's stencil reads a run owned by another machine as that machine's: not stalled, not landed, not to be resumed or cleaned up here, with `lyx adopt` as the takeover.
Rationale: issue #741's orch read a missing pair as a landed run, and its tender status listed nothing.

### Added after Discussion (parent, 2026-10-22): PATTERN.md must not grow for PATTERN-owner-machine

Decision: PATTERN.md must not grow: the PATTERN-owner-machine entry line in PATTERN.md is a short form under the line cap, its full declared text lives verbatim in pattern/PATTERN-owner-machine.md, and the bytes the new line adds are cut from the wording of other PATTERN.md entry lines without changing any rule they state.

Rationale: PATTERN.md is 15703 bytes at the merge base, above pattern.HeadroomBytes (15564), so the per-card headroom lint fails any growth; the declared entry text (about 900 chars) is over pattern.MaxEntryLineChars (360).

## Constraints

- `PATTERN-origin-reconcile`, `PATTERN-owner-machine`: the owner verdict that gates a resume, a wake or a landing against origin is computed inside `ReconcileOrigin`.
  Every other owner read reads the local Bolt: a start's read after its pull and the post-push re-reads of a start, a resume and adopt (D3, D5), Finalize's parent read from the Bolt the landing reconcile left (D7), adopt's `--all` selection (D5), a resume with origin unreachable (D3), and the status views (D9).
- `PATTERN-cli-cobra`, `PATTERN-sandbox-coverage`: `lyx adopt` is one cobra subtree under the root, and the sandbox suite exercises it or excludes it with a reason.
- `PATTERN-leaf-packages`: the machine-name package imports only stdlib.
- `PATTERN-tolerant-records`: `owner_machine` and the hub owner record are tolerant JSON, and a record an older binary wrote reads as unowned.
- `PATTERN-told-geometry`: InnerRun's owner reconcile and tender pause come in through seams.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every refusal and halt above names a way forward an agent's settings allow, and a test reaches it.
- `PATTERN-friction-capture`: D7's halt goes through the halt path that writes a note.

## Auto-mode assumptions

Every decision above is an autonomous pick; the support log's question ledger lists each with its decision.

## Open risks

- Two machines with the same hostname read as one owner, and a renamed host refuses its own runs until it adopts them (D1).
- Two machines that claim an unowned prime while both are offline both act until one push wins (D2).
  A start refused by its post-push re-read leaves its claim's run status on the other machine's entry, as a failed Worktree-Create leaves its claim, and a stopped run record here, which `lyx adopt <slug>` would resume as a takeover of the run the other machine started (D3).
- A resume with origin unreachable trusts the local Bolt, so it can drive a run adopted elsewhere since this machine's last fetch, beside the adopter's driver, until D6's next check after a sleep or a pause hold reaches origin, which may never come before Finalize; only Finalize's landing reconcile is certain to catch it (D3, D7).
  Either side's task-branch push can be the one refused, the adopter's included when this machine pushed first, and each refusal is recorded as a failed push.
- When the adopter lands a run first that the old machine's Finalize had already merged into its local parent, the repair of that parent is a manual `git reset --keep` to origin's tip, or a `git rebase --onto` dropping the squash alone when the parent holds another unpushed commit, which no `lyx` verb performs (D6).
  The old machine's pair is then never torn down by a reconcile, since `finalizeEntered` keeps the slug out of its ended set, and the step 2b records commit is replayed onto origin's parent records branch, publishing the old machine's records of a run origin already ended (D6).
  After `lyx fabric remove <slug>` there, the old machine's batten run record stays not done and its owner pause stays standing, so the orch reads a stopped run that no verb here ends, until `run-process-ownership` (D6).
  A Finalize past its parent-side merge that re-enters through InnerRun's `Spawn` seam once its entry is owned elsewhere is paused, not halted, so no halt note names that repair, and only the local parent's log shows the stale squash (D7).
- A pair under a hub pause reads not live in adopt's step 1, so `--drop-local` there can move a branch under driver strands that are alive though interrupted, as any hub-paused pair's branches move in a reconcile today (D5).
- A run an older binary started has no owner until its next batten resume, so two machines can both drive it until then (D2).
- A child holding the parent lock on a run with no live batten is not held by a slug owner pause, since only its batten releases that lock (D6).
  Nor is one whose slug an unlabelled pause already covers when the wake or `lyx orch start` judges it, since no owner pause is added; its lock is released only at D6's next sleep or hold-end check.
- A driver live on the old machine between adoption and its next owner check pushes into origin's moved branches, and those pushes are refused and recorded as failed pushes; an arbiter or tackler settling a stop there in that window is not checked.
- The old machine's paused run inherits the pause's known leaks, a verify re-prompt and parked pushes (`tender-pause-holds-reprompt`, `run-process-ownership`).
- A run adopted mid-Webster batch meets the machine-local transcript refusals in `internal/websterengine`, which the batch's existing recovery handles.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - adopt-back-replays-old-owner-commits (class design, severity MEDIUM, open in rounds 1, 2, 3, 4)
  - d6-adopter-first-repair-one-sided (class design, severity MEDIUM, open in rounds 3, 4)
  - d6-finalize-running-detector (class decision, severity MEDIUM, open in rounds 4)
  - owner-pattern-text-wider-than-design (class consistency, severity BLOCKING, open in rounds 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-3-review.md; fixer report: _lyx/reviews/plan/round-3-fixer-report.md.
  - setowners-pull-refusal-inputs-unnamed (class consistency, severity MEDIUM, open in rounds 3)
  - setowners-push-transport-row-missing (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- An empty hostname is an error, and no owner is written or compared with it.
- `lyx batten run <slug>` and `lyx loom start` on an existing run, on a machine that does not own the entry, refuse before they create or resume anything, naming the owner and `lyx adopt <slug>`; this covers issue #741's `lyx fabric add` then `lyx batten run --window` path.
- Any InnerRun spawn of `lyx loom start --no-attach` on an existing run refused by the owner check, the resume of an awaiting child included, pauses the pair with one notice, and fails no row.
- A start records this machine as the task's owner, a batten resume of an unowned run-held task records it before resuming, and a hub with no prime owner records the first acting machine.
- A start or batten resume whose owner write the replay lost refuses, naming the owner and `lyx adopt <slug>`.
- A start or an owner-writing batten resume with origin unreachable is refused naming the re-run, a resume of a task the local Bolt shows as owned here warns and resumes, and one it shows as owned elsewhere is refused naming the owner.
- `lyx board upsert` refuses a changed `owner_machine` and accepts the effective one unchanged, and a run-held entry's owner changes through adopt.
- A board replay of two owner writes keeps origin's value and reports the local one lost.
- `ReconcileOrigin` reports owned_elsewhere with the owner name in hub and run scope, judge-only included, with ended taking precedence, and replays no records branch of an owned_elsewhere entry; its `Summary` names each owned_elsewhere entry with its owner.
- The hibernate wake and `lyx orch start` pause an owned_elsewhere run that has a local pair, and name the owner; cleanup only reports it.
- `lyx adopt <slug>` on a second clone reconciles, adds the missing pair, records the owner and pushes Bolt, lifts that slug's owner pause and resumes its run; a refused push, or an owner the replay kept elsewhere, resumes no run, and a named task with no run record gets its owner and no run.
- `lyx adopt <slug>` adopting back a slug with local commits made after an earlier handover records no owner and replays nothing, naming each branch and its ways forward; with `--keep-local` it records the owner, and with `--drop-local` it archives each local tip under a local tag, moves the branch to origin's tip and records the owner, pushing nothing of the dropped commits.
  When the old run's driver strands are still alive under its owner pause, adopt's own `LivePair` call, over seams whose `HubPaused` covers hub entries only, reads the pair live whatever state the reconcile judged its branches, and stops the slug the same way; `--keep-local` records the owner, and `--drop-local` refuses, naming `--keep-local`.
  A test reaches that refusal by handing `SeamOptions.HubPaused` a hub-only cover over a pair whose driver strand is alive under a slug-scope entry and whose code branch holds local commits origin lacks.
- `lyx adopt <slug>` leaves a pause entry without the owner label standing and names it with `lyx resume <slug>`.
- `lyx adopt <slug>` on the parent's owner resumes a run halted at the owner halt, which then lands; on a parent's owner that drove the run before and holds local commits or a live pair for it, the adopt stops at step 1 and its `--keep-local` or `--drop-local` form resumes the run.
- `lyx adopt --prime` records the prime's owner only; every task with an owner, and every run-held task, keeps its owner, and a child halted at the owner halt whose run this machine owns resumes.
- `lyx adopt` refuses an unknown slug, a note, a done task, an unreachable origin and a stop on Bolt, naming each way forward, and `--all` passes over a done task.
- A live run whose entry becomes owned elsewhere pauses through tender after a detected sleep and after a pause hold ends, with one notice.
- InnerRun with origin unreachable after a sleep retries at a later check and holds nothing.
- A run holding the parent lock whose entry becomes owned elsewhere has its lock released and is held by the slug pause, unless its Finalize is running, or awaits its one transient re-step, and already merged into the local parent, which D6's carve-out criteria below cover.
- A run halted at Finalize and adopted by the parent's owner leaves no parent lock held on the halting machine after that machine's next owner check.
- A run whose Finalize merged into the local parent before its entry became owned elsewhere keeps its lock and is not held by the owner pause while Finalize runs, a transient parent-push failure's re-step included; when it pushes first, the landing reaches origin, the adopter's run ends without a second landing, and `lyx resume <slug>` lets the old machine's run go on to Land; and when the adopter lands first, its parent push is refused as a failed push, Finalize stops, and the lock is released at the next pause read.
  That halt note lists the local parent commits origin's parent lacks and names the reset of the local parent to origin's tip when the task's squash is the only one, and the `git rebase --onto` dropping the squash alone when another commit is listed, each with the squash's SHA, then `lyx fabric remove <slug>`, and leaves the owner pause standing;
  a parent push refused while origin's freshly fetched parent lacks the task's work halts with today's reason and names no repair.
- A run stopped at Finalize whose local parent already holds the task's work, with its entry owned elsewhere, has its parent lock released at InnerRun's next pause read and is held by the owner pause.
- A Finalize the shed driver re-steps in-process after its parent-side merge, the re-step after a transient parent-push failure included, with its entry owned elsewhere or origin unreachable at its landing reconcile, does not halt at the owner checks or the unreachable-origin halt and goes on to the push;
  a re-entry of such a Finalize through InnerRun's `Spawn` seam meets D3's owner refusal and pauses the pair.
- Finalize's both-fail halt names `lyx adopt <slug>` on the parent's owner and `lyx adopt --prime <slug> --keep-local` here, and either resumes the halted run; its run-not-owned halt names `lyx adopt <slug> --keep-local` here, which resumes it, and a code branch both machines moved stops at that resume's reconcile with its way forward.
- Finalize halts with a halt note when this machine does not own the run or its parent, or origin is unreachable, naming the ways forward for the check that failed, and lands when it owns both; a Finalize whose local parent already holds the task's work skips both checks and lands.
- A Finalize that halted at an owner check before its merge, then resumed here, meets both checks again.
- A resume past Open-Window opens the window when the switch is on and no client is attached here, a same-machine resume after a detach included, and opens none while one is attached.
- `lyx board list`/`get`, `lyx batten status` and `lyx tender status` show the owner, the README's Running rows name it, and tender lists a run owned elsewhere that has no local pair.
- The orch stencil reads a run owned elsewhere as that machine's.

## PATTERN changes

- `PATTERN-owner-machine` — adds: Starting, resuming or landing a run, bar the rest of a Finalize already past its parent-side merge, or acting on an entry after a wake: only on a machine that does not read the entry as owned by another, where a run-held entry with no owner recorded, and every entry of a hub with no prime owner recorded, reads as owned by no other machine, and any other entry with no owner recorded reads as owned by its parent's owner; its owner written only by `boardengine`'s owner setter; the owner verdict that gates a resume, a wake or a landing judged against origin inside `ReconcileOrigin`, a start's read from the Bolt it just pulled, the re-read after an owner write's push from the Bolt it just pushed, the landing's parent-owner read from the Bolt its reconcile left, and a resume with origin unreachable reading the local Bolt; this machine's name only from `internal/machinename`.

# Decision record — machine-handover

## Goal

Two machines of the operator work on one hub, never at once, and one takes over what the other was doing with one explicit verb and no tangle.
Every task and the prime record an owner machine on Bolt; a machine starts, resumes and lands only what it owns, `lyx adopt` moves ownership here, and a machine that finds a run it drove now owned elsewhere pauses it instead of driving it on.
This closes issues #741 (no ownership, unchecked resume, a missing pair read as landed) and #742 (Open-Window counted done across machines).

## Scope

In:

- This machine's name, and an owner machine per task and for the prime, recorded on Bolt and shown by `lyx board`, `lyx batten status` and `lyx tender status`.
- The owner check on `lyx batten run` (start and resume) and on the shed resume verbs, and the refusal on a machine that does not own the entry.
- `lyx adopt <slug>...`, `lyx adopt --prime` and `lyx adopt --all`.
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

- Before a start, batten reads the entry's effective owner from Bolt after the board's pull of its claim.
  Before a resume, it takes the owner verdict from its run-scope origin reconcile (D4).
- The shed resume verbs on an existing run (`lyx loom start`, `lyx loom resume` and their darn counterparts) take the same verdict from a judge-only run-scope reconcile.
- When another machine owns the entry, the verb refuses before it creates, seeds, spawns or resumes anything, naming the owner and `lyx adopt <slug>` from this machine's prime as the way forward.
- A start records this machine as the task's owner in the claim's Bolt commit.
  A resume of a task with no owner recorded writes the owner in its own Bolt commit and push before it resumes; a refused push refuses the resume.
- An owner write needs its push: a start or an owner-writing resume with origin unreachable is refused, naming a re-run once origin answers.
  A resume of a task this machine already owns keeps today's behaviour with origin unreachable: a Warn, the resume, and the local Bolt copy read.
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
  - the hibernate wake gives each owned_elsewhere entry with a local pair a tender pair pause, as it does a stopped run, and its notice names the owner and `lyx adopt <slug>`;
  - `lyx orch start` gives each owned_elsewhere entry with a local pair a tender pair pause, reports it, and still never refuses;
  - batten's resume and the shed resume verbs refuse as D3 says;
  - the landing rows' run-scope reconcile seam passes the verdict to Finalize (D7), and Merge-In, Publish, PR-Align and PR-Gate proceed on it;
  - `lyx cleanup` only reports it, and tears no such pair down.
- The pause entries these callers and D6 add carry a caller marking them as owner pauses, by the same identity the hibernate daemon gives its owned entry, so `lyx adopt` lifts exactly those (D5).
Rationale: the board entry assigns this outcome to this task, and the reconcile is the one judge every wake and start already runs (`PATTERN-origin-reconcile`).

### D5. `lyx adopt` takes over exactly the entries it names

- A new top-level module (working name `internal/adoptcli`), a cobra root child with no engine of its own, calling each owning module's function in-process, as `lyx cleanup` does.
  It runs from the hub's prime only, by batten's prime refusal.
- Arguments: one or more task slugs, `--prime`, or `--all`, which names the prime and every task that has an owner or a run status.
  A slug that is unknown, a note or done is refused before anything runs.
  `--prime` writes the prime's owner only, so every task with an owner keeps it, and only the unowned tasks no run holds follow the prime here.
- Steps, in order:
  1. A moving hub-scope origin reconcile, which brings warp, the prime's records branch, Bolt and every local pair up to date.
     An unreachable origin, a reconcile error, or a stop on Bolt or the prime's records branch refuses the verb with the report and its way forward, and nothing below runs.
  2. Each named task with a pair on origin and none here gets its pair through fabric's add from origin; a refusal of that add, such as a local warp branch with no worktree, stops that slug with fabric's message and way forward, and the other slugs go on.
  3. One Bolt commit records this machine as the owner of every named entry, the prime included when named, and pushes.
     A refused push refuses the verb, and no run resumes.
     After the push adopt re-reads the owners from Bolt, and a slug whose owner does not read as this machine, because the replay kept origin's value, stops with the owner it reads and starts no run.
  4. This machine's owner pauses (D4) over the adopted slugs are lifted by identity through tender; any other pause entry, a hub pause included, stays.
  5. Each adopted task whose batten run record is not done is started as `lyx batten run <slug> --window` starts it, and the run's own driver lock refuses a second driver here.
     A child halted at D7's owner halt is resumed as `lyx loom resume` resumes it; a child halted for any other reason stays halted for the orch.
- The envelope carries the reconcile report, the `mutations` and `partial` pair of a mutating verb, and per slug what happened: pair added, owner recorded, pause lifted, run started or resumed, the error, and a warning when this machine does not own the slug's parent, since that run halts at Finalize here (D7).
- Bound: adopt writes owners without the old owner's help and does not stop a driver live elsewhere; that machine stops at its next owner check (D3, D4, D6, D7).
Rationale: the board design makes takeover one explicit verb, and steps 1 to 3 run before any resume so no run is driven here before origin records this machine as its owner.

### D6. A live run re-checks its owner after a sleep or a pause hold

- Batten's InnerRun row, which already reads sleeps through its Awake seam and the pause record each check, runs a judge-only run-scope origin reconcile when a sleep of at least a minute ends and when a pause hold over the slug ends.
- An owned_elsewhere verdict pauses the pair through a new tender pause seam on the row, with D4's owner marking, so tender interrupts the child's running turns; the row sends one notice naming the owner and `lyx adopt <slug>`, and holds as under any pause.
- An unreachable origin is retried at each later check until origin answers, and the run is not held meanwhile.
Bound: the child can act for up to one InnerRun check interval after a wake before the check runs, and for as long as origin stays unreachable.
Rationale: a plain suspend, unlike `lyx hibernate`, pauses nothing before sleep, and issue #741's Neve drove the runs on the moment it woke; `lyx resume <slug>` on an owned-elsewhere run would otherwise drive it again unchecked.

### D7. A run lands only on the machine that owns it and its parent

- Finalize, the landing row that merges into the parent's local worktree and pushes it, checks before the parent-side merge that this machine owns the run, by the landing seam's verdict (D4), and owns the run's parent (D2), read from Bolt as that reconcile left it.
- A failed check halts the run at Finalize through the normal halt path, so its halt note is written (`PATTERN-friction-capture`).
  The parent lock the run already holds stays held, as a paused run keeps it, and it is local to this machine.
- The message names the owner and the ways forward: `lyx adopt --prime` here, or `lyx adopt <slug>` on the parent's owner, which moves the run there and resumes it (D5 step 5) to land from its branch on origin.
  This rewords the board's "let the owner land the run" onto built pieces, since the owner landing another machine's run without a takeover is beam work; the halting machine's run is then owned elsewhere and paused at its next owner check.
- An unreachable origin at the landing's reconcile halts Finalize the same way, naming a re-run once origin answers.
- The check sits in the landing producer, so loom and darn runs and an alignment agent's approve that lands with no orch turn all pass it.
Bound: Merge-In, Publish, PR-Align and PR-Gate run on a machine that does not own the parent, since they push only the task branch and merge into no parent.
Rationale: the parent-side merge needs the parent's local worktree, which only the parent's owner keeps current.

### D8. Open-Window opens on a machine with no window attached

- A resume whose Open-Window row is done in the run record runs the row's command once before Run-Shed when the row's switch is on (`open_window`, or `--window`) and reed reports no client attached to the pair's reed session on this machine.
- A first run of the row is unchanged.
Rationale: issue #742; the run record travels with the weft, but a terminal window belongs to the machine and the boot that opened it, and an attached client is the live fact a marker would only approximate.

### D9. Every status view shows the owner

- `lyx board list` and `get` carry `owner_machine` on each task, the effective owner when none is set, and the README's Running rows name it.
- `lyx batten status <slug>` carries `owner_machine` and whether this machine owns it.
- `lyx tender status` names each local pair's owner, and lists each run-held entry owned by another machine with no local pair as owned by that machine.
- The orch role's stencil reads a run owned by another machine as that machine's: not stalled, not landed, not to be resumed or cleaned up here, with `lyx adopt` as the takeover.
Rationale: issue #741's orch read a missing pair as a landed run, and its tender status listed nothing.

## Constraints

- `PATTERN-origin-reconcile`: the owner verdict against origin is computed inside `ReconcileOrigin`; the only other owner reads are a start's read after its claim's board pull (D3) and Finalize's parent read from the Bolt the landing reconcile left (D7).
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
- A run an older binary started has no owner until its next resume, so two machines can both drive it until then (D2).
- A driver live on the old machine between adoption and its next owner check pushes into origin's moved branches, and those pushes are refused and recorded as failed pushes; an arbiter or tackler settling a stop there in that window is not checked.
- The old machine's paused run inherits the pause's known leaks, a verify re-prompt and parked pushes (`tender-pause-holds-reprompt`, `run-process-ownership`).
- A run adopted mid-Webster batch meets the machine-local transcript refusals in `internal/websterengine`, which the batch's existing recovery handles.

## Acceptance criteria

- `lyx batten run <slug>` and `lyx loom start` on an existing run, on a machine that does not own the entry, refuse before they create or resume anything, naming the owner and `lyx adopt <slug>`; this covers issue #741's `lyx fabric add` then `lyx batten run --window` path.
- A start records this machine as the task's owner, a resume of an unowned run-held task records it before resuming, and a hub with no prime owner records the first acting machine.
- `lyx board upsert` refuses a changed `owner_machine` and accepts the effective one unchanged, and a run-held entry's owner changes through adopt.
- A board replay of two owner writes keeps origin's value and reports the local one lost.
- `ReconcileOrigin` reports owned_elsewhere with the owner name in hub and run scope, judge-only included, with ended taking precedence, and replays no records branch of an owned_elsewhere entry.
- The hibernate wake and `lyx orch start` pause an owned_elsewhere run that has a local pair, and name the owner; cleanup only reports it.
- `lyx adopt <slug>` on a second clone reconciles, adds the missing pair, records the owner and pushes Bolt, lifts that slug's owner pause and starts its run; a refused push, or an owner the replay kept elsewhere, starts no run.
- `lyx adopt <slug>` on the parent's owner resumes a run halted at the owner halt, which then lands.
- `lyx adopt --prime` records the prime's owner only; every task with an owner, and every run-held task, keeps its owner.
- `lyx adopt` refuses an unknown slug, a note, a done task, an unreachable origin and a stop on Bolt, naming each way forward.
- A live run whose entry becomes owned elsewhere pauses through tender after a detected sleep and after a pause hold ends, with one notice.
- Finalize halts with a halt note when this machine does not own the run or its parent, or origin is unreachable, naming the ways forward, and lands when it owns both.
- A resume past Open-Window opens the window when the switch is on and no client is attached here, and opens none while one is attached.
- `lyx board list`/`get`, `lyx batten status` and `lyx tender status` show the owner, and tender lists a run owned elsewhere that has no local pair.

## PATTERN changes

- `PATTERN-owner-machine` — adds: Starting, resuming or landing a run, or acting on an entry after a wake: only on the machine that owns it, its owner written only by `boardengine`'s owner setter and judged against origin inside `ReconcileOrigin`; this machine's name only from `internal/machinename`.

# Support log — origin-reconcile

## Interview

The session ran autonomously with no advisors; every question below was settled by the writer and is an auto-pick in the ledger.

### Turn 1 — Reading the task and the board

Read the board entry (design agreed with the operator 2026-10-19, issue #737, folded note `preflight-weft-push-sync`) and the entries it touches: `machine-handover`, `fabric-remote-async`, `run-process-ownership`, `umbrella-daemon`, `prod-pointer-per-machine`.
`run-process-ownership` overlaps on one point: it expects `lyx fabric pull` to settle a diverged Bolt or prime weft of stale status commits.
D5 and D9 settle the stale commits of an ended entry, which are set aside, and those that replay empty; a stale status commit of a live entry that conflicts with origin's newer write still stops with a way forward, and that remainder is left to `run-process-ownership` and `machine-handover`.

### Turn 2 — Exploration

fabricengine already holds every piece the reconcile needs to sit beside: ls-remote parsing in `remoteleftover.go` and `cleanupremotewarp.go`, Bolt's pull-first with its seed-commit drop, the detached coalescing push whose rejection is only a Warn (`unpushedOutcome`), and `Fabric.Pull`'s non-fatal weft arm that names a manual `reset --hard`.
`Topology.Remove` always pushes an archive tag, so a landed-elsewhere removal with no push needs a local mode.
Hibernate's daemon notices go through creel as the program `HIBERNATE`, which gives the push notice its shape.

### Turn 3 — Batch 1: scope and placement

- Where does the reconcile live? Recommended and picked: fabricengine, per PATTERN-fabric-git.
- Does every call site reconcile the whole hub? Picked: two scopes, hub and run, same two calls.
- How does the weft rebase of #737 fit the board entry's "diverged: stop"? Picked: a records-side replay runs first, and only its conflict is "diverged".
- Does code ever get rebased? Picked: never.

### Turn 4 — Batch 2: landed elsewhere and call-site behaviour

- Archive tag local or pushed? Picked: local.
- Who removes the run record? Picked: nobody new; the prime weft reaches origin's state and the landing machine's cleanup owns origin's copy.
- What does a stop do at orch start, wake, resume, Merge-In, Finalize, cleanup? Picked per D8.
- Offline? Picked: `unreachable`, carry on.

### Turn 5 — Batch 3: push failures

- Retry shape? Picked: records side fetch-replay-retry once; code side report; auth report at once; transient keeps the existing backoff.
- Where is a failed push visible? Picked: a record under `.lyx`, one creel mail per episode, printed by `fabric status`, `tender status` and `cleanup`.
- Preflight push? Picked: detached.

### Turn 6 — Lens review

Seven forks ran (interactions, existing-paths, bound-breaker, walkthrough, goal-scope, board-fit, leanness); every fork returned a report.
Duplicates are merged; each line names the lenses that raised it.

- Parent and prime branches moved outside the parent lock (interactions, existing-paths, bound-breaker, walkthrough; BLOCKING) — fixed: D4 moves the prime only under the prime lock and its parent lock, the run scope never moves the parent.
- Hub scope moves live runs' worktrees (interactions, existing-paths, bound-breaker; BLOCKING/MEDIUM) — fixed: D4's live outcome, judged never moved.
- Lock-order inversion between the reconcile replay and the push retry, and weft replay without the push lock (interactions, existing-paths, bound-breaker, walkthrough; BLOCKING) — fixed: write lock then push lock for every records branch; the push retry only try-acquires the write lock and defers.
- Push retry replays stale records of an ended or foreign run (goal-scope, board-fit, interactions; BLOCKING) — fixed for ended entries: D5 sets aside ended entries' commits everywhere, from origin's Bolt tip; for a run another machine still drives, kept as an open risk bounded by the `status.json` conflict, which `machine-handover` closes.
- Bolt replay runs before landed-elsewhere is known, and the Bolt carve-out never matched a Bolt file (interactions, bound-breaker, walkthrough; BLOCKING) — fixed: D3 computes the ended set from the fetched origin Bolt first; D5 defines the Bolt carve-out by entry files.
- A Bolt stop leaves runs unjudged and wake lifts the pause (interactions, bound-breaker; BLOCKING/MEDIUM) — fixed: a D3 stop stops every scoped entry, and wake keeps the hub entry.
- Pairs with no board entry torn down as gone (bound-breaker; BLOCKING) — fixed: D6 needs a batten run record.
- Own landing after Finalize's done mark judged landed elsewhere (walkthrough, bound-breaker; BLOCKING/MEDIUM) — fixed: Finalize entered counts as landing here, and the run scope never applies D6 to its own run.
- Run scope tearing down its own pair (interactions; MEDIUM) — fixed: D6 and the PATTERN-batten-bookend constraint.
- Land fails on D9's new error, and the dropped non-fatal rationale (existing-paths, bound-breaker, walkthrough; BLOCKING/MEDIUM/LOW) — fixed: the warp arm still runs, Land deploys and names the stop.
- Pending records committed by Remove after tagging, so lost (existing-paths, walkthrough; LOW/MEDIUM) — fixed: the local tag replaces the pushed tag at Remove's archive step, after the pending-records commit.
- Warp branch with unlanded work unreported (walkthrough; MEDIUM) — fixed: D6 reports Remove's kept reason.
- Hibernate request owns one entry; pair entries have no lifter or way forward (goal-scope, interactions, existing-paths, bound-breaker, walkthrough) — fixed: plain pair entries not owned by the request, notice names `lyx resume <slug>`.
- `lyx resume`, driver revival and parked-driver resume bypass the reconcile (existing-paths; MEDIUM) — fixed: D8 states they are left as they are, with the reason.
- "No second rebase path" false; other push paths (existing-paths; MEDIUM) — fixed: D5 replaces gitrepo's pull-rebase on every fabric push path, D10 lists the paths.
- Failed-push record never cleared after a teardown (goal-scope, interactions, bound-breaker; LOW/MEDIUM) — fixed: teardown and the reconcile remove it.
- Two mails under concurrency (bound-breaker; LOW) — fixed: exclusive create, creator mails.
- Parked push recreating a branch gone on origin (bound-breaker; MEDIUM) — fixed: D10's gone class.
- Push retry refusing on a live run's uncommitted `status.json` and mailing (walkthrough; LOW) — fixed: a dirtiness refusal is a deferral.
- Bolt fetched twice (bound-breaker; LOW) — fixed: D1 reuses the reconcile's fetch.
- Run record not removed, departure unstated (goal-scope; MEDIUM) — fixed: D6 step 2 states the departure and why.
- "Diverged: stop" changed without saying so (goal-scope; LOW) — fixed: D5's rationale states the reconciliation.
- Orch start and unreachable exceptions buried (goal-scope; LOW) — fixed: named in the Goal.
- Fresh start's reliance unstated (goal-scope, bound-breaker; LOW) — fixed: D8 names Merge-In as the bound.
- Folded note's open trace check dismissed (goal-scope; LOW) — fixed: Scope Out says D11 removes the synchronous half of the pair.
- Dobby re-entering `batten run` runs the reconcile in a loop (board-fit; MEDIUM) — fixed as a scope line: `umbrella-daemon` designs dobby's calls; dobby is not built.
- Local teardown mode duplicates `run-process-ownership`; async-failure question; adopt reuse (board-fit; MEDIUM/LOW) — fixed: Scope names each reuse.
- Busy-at-wake dependency on `run-process-ownership`, and `tender-pause-holds-reprompt` weakening the hold (board-fit, interactions; MEDIUM/LOW) — fixed: Open risks.
- Turn 1 overclaimed what `run-process-ownership` can rely on (board-fit; MEDIUM) — fixed in Turn 1.
- Notice hard-coded to the prime's orch against `orch-any-worktree` (board-fit; LOW) — fixed: the mail goes to the orch batten's notices reach.
- "Drives" reads shared prime records as machine-local (board-fit; LOW) — fixed: D6 judges from local records before any move and ignores pairs without a run record; owners make it exact.
- Phantom run and repeated ended reports while origin still holds the record (interactions, existing-paths; LOW) — rejected: resume is refused and the noise ends when the landing machine's cleanup runs; no requirement changes.
- "Gone" reported as landed (interactions; LOW) — fixed: D6 reports done and gone apart.
- Auto-mode assumptions duplicating decisions, perishable measurements, notes duplicating `relevant.md`, duplicated way-forward paragraph, spawn-observability constraint (leanness; MEDIUM/LOW) — fixed: cut.
- Interview turns duplicating the ledger (leanness; LOW) — rejected: the support log is the review's trail and the plan writer never reads it.

## Rejected alternatives

- **Bolt-only check**: misses task-branch commits between status writes, weft commits that change no status and hand pushes such as warp `main` (board entry).
- **A full `git fetch` instead of ls-remote**: downloads every moved branch, where ls-remote answers in one round trip and only moved heads are fetched.
- **Reconcile per shed row**: the operator ruled out a remote check per row or in a loop.
- **`git reset --hard origin/<branch>` on a diverged weft**: discards non-empty local commits; the replay keeps them or stops.
- **Rebasing a diverged code branch**: rewrites work an agent may be mid-way on; code divergence is a human call.
- **Pushing the landed-elsewhere archive tag**: pushes stale records from a machine that did not land the run, which `run-process-ownership` objects to.
- **A removal commit for the run record**: the next push would replay it over origin, and origin's copy belongs to the landing machine.
- **Refusing orch start on a stop**: the orch is the session that resolves the stop.
- **Refusing every call site while origin is unreachable**: blocks all offline work, while D10 still reports the later rejected push.
- **A failed-push record on Bolt**: pushing a record of a failed push needs the push that failed; the record is machine-local.
- **A creel mail per failed push**: a read-only account fails every push, which would flood the orch.
- **Serialising the board sync and the weft push**: unproven cause of the bare `(failed)`, already retried with backoff.

## Review rounds

### Round 1

BLOCKING, 22 findings, every one fixed.
The Bolt replay goes by board entry through an injected boardengine merge, since every Bolt commit rewrites `board.json` and `README.md`.
An entry only local Bolt holds is never ended, and wake judges a paused pair as quiet.
D5 takes the ended set from its caller and never sets aside this machine's own landing.
A cleanup dry run reconciles judge-only, and the prime's locks are tried once, never waited on.
fabricengine reaches teardown, liveness, notice and the board merge through caller seams.
The failed-push record carries a `notified` flag, and a deferral writes a `deferred` record and ends the holder's loop.

### Round 2

BLOCKING, 8 findings, every one fixed.
The wake reconcile's network calls pass its own request's hub pause entry; orch start and cleanup wait at the gate under an operator's hub pause.
Records-branch locks are tried once and a held one reads busy, and the teardown's quiet wait is zero.
Rewound and rewritten read the tracking ref's reflog since this machine's last push to it, and the cleanup dry run fetches into no ref.

### Round 3

BLOCKING, 7 findings, every one fixed.
The Bolt entry merge is a boardengine entry point under the caller's locks, and a refusal sets aside one change, not a commit.
Cleanup's teardown of an ended pair under a wake-written pause passes force for dirtiness and keeps the uncommitted paths under a local tag.
Wake skips its reconcile while a hub entry its request does not own is live, so an operator's hub pause is never passed.
Removal from origin is read from the fetched tip's own history, matched by `created` where local Bolt holds the slug; a rewind of origin's head separates rewound from rewritten.
The local teardown try-acquires the teardown and weft write locks and keeps only the waits for its own loop kill, and a gone push whose entry is not yet done on origin backs off before it records.
The Bolt entry merge applies changes through the board's validation, a refused change is set aside like a field clash, and an added entry takes a fresh id.
The push retry and the gone path fetch Bolt before reading the ended set, and `lyx cleanup --apply` ends the driver of an ended pair under a wake-written pair pause entry.

## Question ledger

- Q1 Reconcile placement — resolved, auto-pick: fabricengine (PATTERN-fabric-git).
- Q2 Scope per call site — resolved, auto-pick: hub scope for orch start, wake and cleanup; run scope for batten resume, Merge-In and Finalize.
- Q3 Weft divergence versus "stop on divergence" — resolved, auto-pick: replay records first, stop only on conflict; code always stops.
- Q4 Archive tag of a landed-elsewhere run — resolved, auto-pick: local tag, reported (namespace superseded by Q22).
- Q5 Run record removal — resolved, auto-pick: no commit of the reconcile's own.
- Q6 Busy driver of a landed-elsewhere run — resolved, auto-pick: reported busy, not torn down, never resumed.
- Q7 Orch start on a stop — resolved, auto-pick: start anyway, report in the envelope and the start note.
- Q8 Wake on a stop — resolved, auto-pick: a plain pair pause entry, not owned by the request, lifted with `lyx resume <slug>` (superseded by Q20).
- Q9 Batten fresh start — resolved, auto-pick: no reconcile; `Add` probes origin already.
- Q10 `lyx cleanup` — resolved, auto-pick: reconcile first; a Bolt or prime-weft stop refuses the verb.
- Q11 Unreachable origin — resolved, auto-pick: warn and carry on.
- Q12 Push failure classes — resolved, auto-pick: per D10.
- Q13 Where a failed push shows — resolved, auto-pick: `.lyx` record, one creel mail per episode, `fabric status`, `tender status`, `cleanup`.
- Q14 Creel sender of the notice — resolved, auto-pick: new program `FABRIC`.
- Q15 Preflight push — resolved, auto-pick: detached.
- Q16 Board sync and weft push race — resolved, auto-pick: no serialisation; existing backoff covers it.
- Q17 Who moves the prime and parent branches — resolved, auto-pick after the lens review: the prime only under the prime lock and its parent lock, the parent at run scope never.
- Q18 Lock order of the push retry — resolved, auto-pick after the lens review: try-acquire the write lock under the push lock, defer when held.
- Q19 How a woken machine knows an ended entry before replaying Bolt — resolved, auto-pick after the lens review: from the fetched origin Bolt tip.
- Q20 Ownership of wake's pair pause entries — resolved, auto-pick after the lens review: plain entries, lifted with `lyx resume <slug>`.
- Q21 Ended set outside the reconcile — resolved, auto-pick in review round 1: `Fabric.Pull` and the push retry read it from the origin Bolt tracking ref their fetch left, with no network call, and the Finalize-entered exclusion applies inside D5.
- Q22 Tag namespace of set-aside commits and the ended weft tip — resolved, auto-pick in review round 1: a local `set-aside/<slug>/<sha>` prefix for both, which no archive reader matches.
- Q23 Source of the push's gone class — resolved, auto-pick in review round 1: one `git ls-remote origin` of the branch before every push of a branch with an upstream.
- Q24 Orch start and `lyx cleanup` under an operator's hub pause — resolved, auto-pick in review round 2: wait at the network gate until the lift, as every network call of the hub does; only the wake daemon's own reconcile passes its own request's entry.
- Q25 Evidence of a rewind once the tracking ref moved — resolved, auto-pick in review round 2: the tracking ref's reflog from this machine's last push to it onward.
- Q26 A dirty worktree of an ended pair whose busy driver cleanup ends — resolved, auto-pick in review round 3: force for dirtiness alone, the uncommitted paths kept under a local `set-aside/` tag and reported; a merge in progress still refuses.
- Q27 Wake under a hub pause the request does not own — resolved, auto-pick in review round 3: the wake reconcile is skipped and the notice says why, since passing an operator's pause breaks Q24; the daemon lifts only an owned entry.
- Q28 Rewound against rewritten — resolved, auto-pick in review round 3: origin's head an ancestor of an earlier tracking-ref value makes it rewound, otherwise rewritten; `lyx fabric pull` refuses a rewind once the reconcile's fetch moved the ref (widened by Q30 to every rewind of a task branch).
- Q29 A gone push whose entry origin's Bolt does not yet show ended — resolved, auto-pick in review round 3: the existing backoff, then the record and mail.
- Q30 How `lyx fabric pull` recovers a rewritten branch the reconcile already fetched — resolved, auto-pick in review round 4: its warp arm's unpushed test reads the tracking ref's reflog as D4 does, and it refuses a rewound task branch, never the prime's, so the reset reaches only commits origin once held.
- Q31 The local teardown's remaining waits — resolved, auto-pick in review round 4: the hub workspace regeneration by try-acquire, no store gc, the snapshot under its own weft write lock try-acquire, and the loop-kill waits summed over a hub's ended pairs.
- Q32 Wake with an unreachable origin — resolved, auto-pick in review round 4: retry both ls-remote calls within a 2-minute bound, then lift as today with an `unreachable` notice naming the reconcile to run; keeping the hub entry live would hold every offline driver after each wake.

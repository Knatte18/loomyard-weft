# Decision record — pawl-orch-landing

## Goal

A beam's landing work moves into Beam-Await, driven by the beam's orch with the operator: the orch writes the description, opens and updates the beam's pull request, brings the parent in through a merge-in run and checks the pull request's merge state.
`lym beam ready` becomes the point of no return.
Pawl drops Parent-Align, Beam-Describe and PR-Review and runs Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down.
Finalize stays the only place a beam merges into its parent, keeps the beam pull request's text as the orch wrote it, and lands a beam with a pull request on GitHub.
Issues #57, #56 and #47 are settled as the board entry folds them in.

## Scope

In:

- The pawl recipe's rows, the hold boundary, and Beam-Await's accept and interrupt handling.
- The verbs `lym beam describe` and `lym beam pr open|update|status|close`, and the new refusals of `lym beam ready`.
- A tracked beam landing record, the dobby's notice to the beam's orch at each member landing while a pull request is recorded, and Finalize's landing of a pawl run with and without one.
- The dobby holding a ready beam: no member run starts once the ready record stands.
- Removing `lym beam reopen`, the reopen request and PR-Review's reroute.
- `lym beam start` bringing back the orch of a beam in landing.
- Moving an in-flight pawl run off a removed row.
- The orch role text for a beam's landing.

Out:

- Tapestry, darn and sampler keep Parent-Align, PR-Review and Finalize's pull-request handling as they are.
- No `lym beam pr merge` verb: the orch never merges a pull request.
- Note `pr-review-wait-silent` stays a task-landing concern; this task removes only the pawl case of it.
- The push lock's locking (see Open risks).

## Decisions

### D1 — Pawl's rows

The pawl recipe (`contracts/recipes/pawl-recipe.yaml`) runs Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down; Beam-Await's `on_done` is Finalize.
The BeamDescribe constructor leaves `internal/shedrecipe`'s registry, since no recipe row runs it; ParentAlign and PRReview stay registered for the other recipes.

### D2 — The hold boundary is Finalize, and a ready beam starts no member

Beam-Await's accept writes the entry status naming Finalize, not Parent-Align, as the landing hold.
`boardengine.RunHolds`' hold-free rows stay Worktree-Create, Preflight, Record-Write, Beam-Up and Beam-Await, so the run holds its beam from Finalize on.
From the ready record on, the beam's dobby starts no member run under the beam.
A member filed after ready stays unstarted and blocks the landing, as any open member does; the dobby posts one notice per episode to the beam's orch naming it and `lym batten cancel <member> --remove`, since ready is final and the member cannot land into this beam.
A failing re-check at the accept still lifts the hold, restarts the dobby and posts the blockers, as today.
Bound: between `lym beam ready` and the accept, the beam is held by the dobby's refusal to start members and by D3's refusal of open work; a member filed in that window never starts, so nothing lands into a ready beam.

Rationale: the entry holds the beam from `lym beam ready` on and moves the `RunHolds` boundary from Parent-Align to Finalize; the board cannot read the tracked ready record, so the dobby that starts members enforces the hold until the status takes over.

### D3 — `lym beam ready` refuses before the point of no return

`lym beam ready` keeps its entry checks and adds these refusals, each naming its way forward:

1. The pawl run is not at Beam-Await, running or halted: names `lym batten status <slug>`.
2. Anything works under the beam, as the landing refusal (`internal/battencli/beamland.go`) reads it, open members and child pairs: names each blocker as that refusal does.
3. The parent holds commits not merged into the beam: the parent pair's local tip, and its upstream tip fetched under the remote-refs lock when the parent has an upstream and the hub's push skip is unset, is not an ancestor of the beam's head; names `lym beam merge-parent <slug>`, or the open merge-in task when one stands.
4. The beam's landing record (D4) names no description written for this beam, or the description fails validation: names `lym beam describe <slug>`.
5. The landing needs a pull request and none is recorded: the resolver answers a required pull request (beam.yaml `required`, or GitHub's rules for the parent) with no pull request in the landing record; names `lym beam pr open <slug>`.
6. A pull request is recorded and either its head on GitHub is not the beam's head, naming `lym beam pr update <slug>`, or D6's merge state is not mergeable-now, naming the cause and `lym beam pr status <slug>`.
   A pull request merged on GitHub passes, since Finalize lands it through the already-landed path; a closed one names `lym beam pr open <slug>`, which reopens it, or `lym beam pr close <slug>`.

A beam already flagged ready reports `already` as today, and no verb removes the ready record: Finalize's landing and Beam-Down end it.
Bound: refusals 3 and 6 read a moment, and after ready only the parent and GitHub can move, since no member starts (D2) and the beam verbs refuse (D5, D6); Finalize merges the parent in again and halts, with a way forward, on a merge GitHub refuses.

Rationale: once ready, there is no way back to Beam-Await, so every condition the operator could still fix with the orch is checked before it.

### D4 — The beam landing record

A tracked beam landing record sits in the beam worktree's landing directory, beside the ready record, through `internal/state`, with its lock under `.lym/locks/` and its paths named only by `internal/lymdirs` and the landing-directory accessor.
It names the beam it was written for, and a record naming another slug reads as no record and is never removed, as the ready record's `ReadyFor` and `RemoveReady` filter, since a pair forked from the beam inherits the beam's records.
It carries:

- the time `lym beam describe` last wrote the description for this beam;
- while a pull request is recorded, its number, URL, the beam head it was last pushed with, and the beam head D7 announces from.

`lym beam describe` and the `lym beam pr` verbs write and commit it; `lym beam pr close` drops the pull request from it.
Finalize reads it through a seam the pawl wiring tells and drops the pull request from it, committed, only after the parent-side landing completes.

Rationale: the record must survive a restart or an adopt onto another machine, since Finalize and the dobby act on it; Finalize's scratch pull-request record keeps serving what Finalize opens itself (D8).

### D5 — `lym beam describe`

`lym beam describe <slug>` writes the beam's change description, `_lym/landing/summary.md`, with the description writer Beam-Describe runs today (entry title and brief, the beam's first-parent commits each with its pull request number or landing commit, merge time and summary, and the project record's added decisions), validates it, stamps D4's record and commits both.
It refuses a beam whose run is not at Beam-Await, running or halted, a beam whose ready record stands, and a description that fails validation, naming the finding and `lym board upsert` for the entry's title or brief.
The writer moves from a producer to a function the verb calls; it stays in `landingshed`.
Nothing at landing rewrites `summary.md`.

Rationale: issue #56's landing rewrote the orch's text; the description is now written only when the orch asks.

### D6 — `lym beam pr open|update|status|close`

Thin verbs under `lym beam pr`, over `internal/pullrequest`, called by the beam's orch.
`open`, `update` and `close` refuse a beam whose run is not at Beam-Await, running or halted, whose ready record stands, whose pair is not here or which another machine owns, each with the way forward `land`'s checks name today.
`status` is read-only and answers at any row up to Beam-Down.

- `open`: refuses a description D4 does not stamp for this beam (names `lym beam describe`), a beam with no commits beyond its parent, and an origin the resolver's origin check refuses (no GitHub origin, no parent upstream, push skip set).
  It pushes the beam branch through fabric, finds an open pull request from the beam branch into its parent or creates one with the description's title and body, marks a found draft ready for review, and records it in D4 with the pushed head as both its head and the head D7 announces from.
  A recorded pull request that GitHub reports open answers `already`; one GitHub reports closed is reopened.
- `update`: pushes the beam branch, sets the pull request's title and body from the description, marks a draft ready for review, and records the pushed head; refuses with no pull request recorded, naming `lym beam pr open`.
- `status`: reads the recorded pull request from GitHub and prints its state, draft, head, `mergeable_state`, review approval and each required check with its conclusion, plus `mergeable_now` and the named causes when it is false.
  `mergeable_now` is true only for an open, non-draft pull request whose head is the beam's head and whose `mergeable_state` is `clean`, `has_hooks` or `unstable`; `dirty` is a conflict, `blocked` a review or required check, `behind` a base ahead of the head, and `unknown` is retried for a short bounded window and then reported as not yet computed; a closed or merged pull request is named as such.
  D3's refusal 6 calls the same function.
- `close`: closes the recorded pull request on GitHub with a comment naming the operator's choice to land without one, skipping the GitHub call for one already closed or merged, and drops it from D4.

The orch never merges a pull request, and never closes one to add work: work lands into the beam while it stays open (D7).
Bound: the verbs open, edit, reopen and close only the pull request from the beam's own branch into its parent, read a token only through `internal/githubclient`, and change nothing once the ready record stands.

Rationale: the orch calls lym verbs, never `gh` by hand; the GitHub code is the code PR-Review and Finalize run today.

### D7 — Member landings while the pull request is open

While D4 records a pull request, the beam's dobby posts one notice to the beam's orch for each landing into the beam branch, members filed after the pull request opened included, with no `notify_on_land` on any entry.
The dobby reads the beam branch's first-parent commits past the last head it announced, and posts one line naming them and `lym beam describe <slug>` then `lym beam pr update <slug>`.
The last announced head is never-tracked delivery state under the beam worktree's `.lym/beam/`; with it missing, the dobby announces from D4's announce head, so a fresh machine repeats at most one notice.
Bound: a notice that fails to post is retried at the next pass, and nothing else depends on it; a member that also carries `notify_on_land` gets that notice too.

Rationale: the operator keeps landing members into a beam whose pull request is open, and the orch keeps the description listing every member.

### D8 — Finalize lands a pawl run by the beam's record

For a pawl run, Finalize's pull-request decision reads D4's record beside the resolver's inputs, through `landingshed.ResolvePullRequest`:

- A pull request is recorded: the landing goes through it, always by squash.
  Finalize takes the parent lock, merges in what the parent gained and runs the landing gate as today, pushes the beam branch, squash-merges the recorded pull request on GitHub with the description's title and body as the commit, pulls the parent, completes the parent-side landing, and only then drops the pull request from D4.
  It never edits the pull request's title or body.
  A pending merge past the retry window halts naming the check to settle on GitHub and `lym loom resume`; a recorded pull request GitHub reports closed halts naming the operator reopening it on GitHub and `lym loom resume`; one already merged lands through the already-landed path.
- None recorded, and a pull request is required (beam.yaml `required` or `merge`, or GitHub's rules): Finalize finds an open pull request from the beam branch, leaving its text as it is, or opens one from the description, and merges it, as `protected-parent-landing` built for `merge`.
  A merge GitHub refuses, such as a required review, halts naming what GitHub refuses, `lym beam pr status <slug>` and `lym loom resume`.
- None recorded otherwise (beam.yaml `on` or `off`): Finalize squash-merges into the parent and pushes, as today; a push the parent refuses as protected halts with the way forward `protected-parent-landing` gives.

For a beam, the entry's `pull_request` field is not read: whether the beam has a pull request is the orch's act, recorded in D4.
Rationale: the operator chooses with the orch whether a beam lands through a pull request; a parent that demands one keeps demanding it, and every halt names a step that works at Finalize.

### D9 — Finalize keeps a beam pull request's text

In a pawl run, Finalize never edits the title or body of a pull request it finds or has recorded; only a pull request Finalize creates gets the description's text.
The squash commit's title and message still come from the change description, with the pull request's number and the credit block, as today.
Other recipes keep today's behaviour.
Rationale: issue #56.

### D10 — No way back from ready

`lym beam reopen`, the reopen request, Beam-Await's handling of it and PR-Review's reroute to Beam-Await go, with PR-Review's `on_reroute` row in pawl and PR-Review's reopen handling, which only pawl used.
Bound: `lym loom goto` stays the operator's hand escape; it leaves the ready record standing, so the dobby lands the beam again once nothing works under it, and it adds no way to take new work.

### D11 — Beam-Await survives a loop exit

Beam-Await, Beam-Up and Beam-Down get the re-invoke interrupt policy, since each spawns no agent and is idempotent.
The driver's rules re-step a re-invoke stop (`internal/sheddriver/driverules.go`), and the arbiter's eligibility for an interrupted stop under the re-invoke policy extends from tapestry, darn and sampler to pawl, so a loop exit at Beam-Await past the driver's repair cap is still resumed with no hand `lym loom resume`.
A loop exit after a wake that completed counts as a new stop for the repair cap, so a beam waiting for days survives every exit.
A re-entered Beam-Await finding the landing hold with no land request, a crash between the accept's removals and Done, starts the dobby when the dobby-stop marker stands, as `refuse` does, and the dobby writes the land request again from the tracked ready record.
That entry check reads the hold by the Finalize name or a retired row name (D13), so a crash under the old binary is caught too.
Rationale: issue #57.

### D12 — `lym beam start` in landing

`lym beam start` on a beam its own pawl run holds, Finalize and Beam-Down included, brings back the beam's orch and leaves its dobby down, reporting that the dobby stays down while the landing runs; the refusal (`internal/orchcli/refusal.go`) stays for a beam held by anything else.
Bound: `lym beam start` never starts the dobby of a held beam; Beam-Down still takes the orch down.
Rationale: issue #47; the landing tolerates a live orch, and the operator needs it to follow a halted Finalize.

### D13 — In-flight pawl runs on a removed row

A recipe may declare retired rows, each mapped to the row a run on it moves to; pawl declares Parent-Align, Beam-Describe and PR-Review, each to Beam-Await.
Shed's step, run by a new-binary process and finding a status whose current row is retired, moves it once under the status lock: the status names the target row, halted with a reason naming the move and `lym loom resume`; the entry's board status names the target too, lifting the hold.
The move's halt is judged first, by its row and reason, as the parent's and never the tackler's or the arbiter's, so the run stays paused for the orch.
For pawl the move also:

- ends a live PR-Review session of the run through `runproc`;
- carries an open pull request from the run's scratch record into D4's record, with the beam's head as its announce head, and removes the scratch record;
- removes the ready record, the land request and the dobby-stop marker;
- posts one notice to the beam's orch: the run was moved to Beam-Await and paused, the pull request stays open, run `lym beam start <slug>` and `lym loom resume`, a second resume when the move was made by a resume, then the new verbs.

A run at Finalize or past it is untouched.
Bound: only a status naming a row the recipe lists as retired moves; any other unknown row keeps today's refusal naming `goto`.
A driver process of the old binary still alive keeps running the old rows until it stops; the move happens at the first step a new-binary process makes.
Rationale: the row lookup refuses a status naming a row the recipe lacks today, and no row-migration mechanism exists; a recipe-declared map keeps it in shed's shared machinery.

### D14 — The beam orch's landing instructions

The orch role (`contracts/orchrole/orch-role.md`) carries a beam's landing workflow: describe, open, update and check the pull request when the operator says so, merge the parent in through `lym beam merge-parent`, act on the member-landing notices, and run `lym beam ready` only when the operator is satisfied, knowing it is final.
The `ready`, `pr` and `describe` verbs' help texts state the same, and the removed `reopen` text goes.

## Constraints

- A flock is released when every descriptor of it is closed, and Go opens every file close-on-exec (Go `os` package docs; flock(2) man page), so a push lock whose holder process died is free already.
- GitHub's REST pull request object carries `mergeable` and `mergeable_state`, whose values are `clean`, `dirty`, `blocked`, `behind`, `unstable`, `has_hooks`, `unknown` and `draft`, mirroring the GraphQL `MergeStateStatus` enum (GitHub GraphQL schema, `MergeStateStatus`); GitHub computes mergeability asynchronously and reports unknown until done. D6 reads the REST fields through go-github, which `internal/pullrequest` already uses.
- `PATTERN-pull-request-mode`: the resolver stays the one decision; D8 adds the beam record as an input (PATTERN changes).
- `PATTERN-github-auth`: the `lym beam pr` verbs join the callers that open, reopen and close a pull request (PATTERN changes).
- `PATTERN-remote-refs-lock`: D3's parent fetch and D6's pushes go through `gitrepo` and fabric under the lock.
- `PATTERN-shed-shared-machinery`, `PATTERN-sole-parsers`, `PATTERN-shed-stop-chain`: D13's retired-row map is read by the recipe's sole parser and applied in shed's packages, and its halt is judged first as the parent's.
- `PATTERN-tracked-file-locks`, `PATTERN-durable-vs-ephemeral-state`, `PATTERN-tolerant-records`: D4's record is tracked with its lock under `.lym/locks/`; D7's announced head is never-tracked delivery state.

## Auto-mode assumptions

- "The hub's rules for beam pull requests" reads as the resolver over the parent's beam.yaml, GitHub's rules and the origin facts (D6, D8).
- The orch skips a pull request by not opening one or closing it (D8).
- No `lym beam pr merge` verb (D6).
- `lym beam ready` refuses more than the two causes the entry lists (D3).

## Open risks

- Issue #47's push lock: the entry asks that "a push lock whose holder died before a restart is taken as stale", but such a lock is already free (Constraints); the wait #47 saw was a live holder, likely a push the restart left running. This record changes nothing there. Question for the parent: should lym stop a live push left from before a restart, and by which rule, or does this point drop?

## Acceptance criteria

- D1: the pawl recipe builds with the rows Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down, Beam-Await's `on_done` is Finalize, and no recipe row names BeamDescribe.
- D2: an accepted land request writes the entry status naming Finalize; `boardengine.RunHolds` reads a pawl run at Beam-Await as holding nothing and at Finalize as holding the beam; a beam dobby with the ready record standing starts no member run and posts the open-member notice once; a failing re-check writes Beam-Await back and restarts the dobby.
- D3: `lym beam ready` refuses, each with its way forward, a run not at Beam-Await, open work under the beam, a parent tip not an ancestor of the beam head (naming `lym beam merge-parent`), a parent with no upstream judged by its local tip alone, no description stamped for this beam, a required pull request with none recorded, a recorded pull request whose head differs from the beam's (naming `lym beam pr update`), and one not mergeable now or closed; a merged recorded pull request passes; it writes the ready record when none applies.
- D4: a landing record naming another slug reads as no record and is not removed; describe and `pr open` write and commit a record naming the beam; Finalize drops the pull request only after the parent-side landing.
- D5: `lym beam describe` writes and commits a description listing each beam commit with its pull request number or commit, merge time and summary, stamps the record, and refuses a run not at Beam-Await and a beam whose ready record stands; a halted run at Beam-Await is accepted.
- D6: `open` adopts an open pull request without editing it except a draft flag, creates one from the description otherwise, reopens a closed recorded one and answers `already` for an open recorded one; `update` sets title and body and the recorded head; `status` reports `mergeable_now` true only for an open non-draft pull request at the beam's head in `clean`, `has_hooks` or `unstable`, and names the cause otherwise, closed and merged included; `close` closes it, or skips GitHub when it is closed or merged, and drops it from the record; `open`, `update` and `close` refuse a run not at Beam-Await and a beam whose ready record stands, and `status` answers at Finalize.
- D7: with a pull request recorded, a member landing into the beam produces one notice to the beam's orch, a later pass with no new commit posts none, a member filed after the pull request opened is announced too, and a missing announced-head marker announces from the record's announce head; with none recorded, no notice.
- D8: a pawl Finalize with a recorded pull request squash-merges it on GitHub without editing its text, whatever `Config.Squash` says, and drops it from the record after the parent-side landing; a crash between the GitHub merge and the parent-side landing re-enters through the already-landed path; with none recorded and a required or `merge` parent it finds or opens a pull request and merges it, halting on a GitHub refusal with `lym beam pr status`; with none recorded and `on` or `off` it lands by a local squash merge and push.
- D9: a pawl Finalize finding an open pull request merges it without a title or body edit, and a task run's Finalize behaves as today.
- D10: `lym beam reopen` is no longer a verb, and no pawl row reroutes to Beam-Await.
- D11: Beam-Await, Beam-Up and Beam-Down carry the re-invoke interrupt policy; an interrupted pawl stop at Beam-Await under it is eligible for the arbiter; a second loop exit after a completed wake is re-stepped; a Beam-Await entry finding the hold named Finalize or a retired row with no request starts the dobby when the dobby-stop marker stands.
- D12: `lym beam start` on a beam held by its own pawl run at Finalize starts its orch and not its dobby, and still refuses a beam held otherwise.
- D13: a pawl status naming Parent-Align, Beam-Describe or PR-Review is moved to Beam-Await, halted with a stop routed to the parent, with the board status naming Beam-Await, the scratch pull-request record carried into the landing record and removed, the ready record, land request and dobby-stop marker removed, and one notice posted; a status naming a row neither present nor retired still refuses naming `goto`; a run at Finalize is untouched.
- Review: D14's orch role text describes the beam landing workflow and the finality of `lym beam ready`, and no text names `lym beam reopen`.

## PATTERN changes

- `PATTERN-pull-request-mode` — changes: Deciding if a landing uses a pull request: only landingshed's one resolver, from the receiving `beam.pull_request`, the entry's `pull_request` or a beam's pull-request record, the parent's GitHub rules, origin and task commits; no run setting carries it; a yes merges on GitHub.
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize, `lym beam pr` and pair teardown open, reopen, merge or close a pull request; `lym beam pr` never merges. (test)

## Notes for the plan writer

- D13's retired-row map is a recipe-level key read by the recipe parser; the pawl side effects (session end, record carry, file removals, notice) hang on a seam the pawl wiring tells, so shed stays recipe-agnostic.
- D7 fits beside the dobby's existing landing notice (`internal/dobbyengine/landnotice.go`), which posts through the same nearest-orch path; D2's start hold fits the dobby's start step beside its ready read (`internal/dobbyengine/land.go`).

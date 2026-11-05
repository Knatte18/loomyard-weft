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
- The dobby and the member fork holding a ready beam: no member run starts once the ready record stands.
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
From the ready record on, no member run starts under the beam.
The beam's dobby starts none, and a member run's fork precondition, the `boardengine.RunHolds` check in battencli's beam fork (`internal/battencli/beamfork.go`), also refuses a member whose beam's ready record names that beam, naming `lym batten cancel <member> --remove`, as the dobby's notice does; a ready record the fork cannot read refuses too, naming the error.
A member filed after ready stays unstarted and blocks the landing, as any open member does; the dobby posts one notice per episode to the beam's orch naming it and `lym batten cancel <member> --remove`, since ready is final and the member cannot land into this beam.
A failing re-check at the accept still lifts the hold, restarts the dobby and posts the blockers, as today.
Bound: between `lym beam ready` and the accept, the beam is held by two guards, the dobby's start hold and the fork precondition, and by D3's refusal of open work; a member filed in that window never starts, by the dobby or by a hand `lym batten run`, so nothing lands into a ready beam.

Rationale: the board cannot read the tracked ready record, so the dobby that starts members and the fork every member run passes enforce the hold until the status takes over.

### D3 — `lym beam ready` refuses before the point of no return

`lym beam ready` keeps its entry checks and adds these refusals, each naming its way forward:

1. The pawl run is not at Beam-Await, running or halted: names `lym batten status <slug>`.
2. Anything works under the beam, as the landing refusal (`internal/battencli/beamland.go`) reads it, open members and child pairs: names each blocker as that refusal does.
3. The parent holds commits not merged into the beam: the parent pair's local tip, and its upstream tip fetched under the remote-refs lock when the parent has an upstream and the hub's push skip is unset, is not an ancestor of the beam's head; names `lym beam merge-parent <slug>`, or the open merge-in task when one stands.
4. The beam's landing record (D4) names no description written for this beam, or the description fails validation: names `lym beam describe <slug>`.
5. The landing needs a pull request and none is recorded: the resolver answers a required pull request (beam.yaml `required`, or GitHub's rules for the parent) with no pull request in the landing record; names `lym beam pr open <slug>`.
6. A pull request is recorded and either its head on GitHub is not the beam's head, naming `lym beam pr update <slug>`, or D6's merge state is not mergeable-now, naming the cause and `lym beam pr status <slug>`.
   A pull request merged on GitHub passes when the merge lacks no beam commit: its merged head is the beam's head, or the beam's first-parent commits past it only merge the parent in; Finalize lands it (D8).
   One whose merge lacks beam commits is refused, naming those commits and the way forward: `lym beam pr close <slug>`, then land without a pull request or open a new one with `lym beam pr open <slug>`.
   A closed one names `lym beam pr open <slug>`, which reopens it, or `lym beam pr close <slug>`.

A beam already flagged ready reports `already` as today, and no verb removes the ready record: Finalize's landing, Beam-Down and D13's move end it.
Bound: refusals 3 and 6 read a moment, and after ready only the parent and GitHub can move, since D2's guards start no member and the beam verbs refuse (D5, D6); Finalize merges the parent in again and halts, with a way forward, on a merge GitHub refuses.

Rationale: once ready, there is no way back to Beam-Await, so every condition the operator could still fix with the orch is checked before it.

### D4 — The beam landing record

A tracked beam landing record sits in the beam worktree's landing directory, beside the ready record, through `internal/state`, with its lock under `.lym/locks/` and its paths named only by `internal/lymdirs` and the landing-directory accessor.
It names the beam it was written for, and a record naming another slug reads as no record and is never removed, as the ready record's `ReadyFor` and `RemoveReady` filter, since a pair forked from the beam inherits the beam's records.
It carries:

- the time `lym beam describe` last wrote the description for this beam;
- while a pull request is recorded, its number, URL, the beam head it was last pushed with, and the beam head D7 announces from.

`lym beam describe` and the `lym beam pr` verbs write and commit it; `lym beam pr close` drops the pull request from it.
Finalize reads it through a seam the pawl wiring tells and drops the pull request from it, committed, only after the parent-side landing completes.

Rationale: the record must survive a restart or an adopt onto another machine, since Finalize and the dobby act on it.

### D5 — `lym beam describe`

`lym beam describe <slug>` writes the beam's change description, `_lym/landing/summary.md`, with the description writer Beam-Describe runs today (entry title and brief, the beam's first-parent commits each with its pull request number or landing commit, merge time and summary, and the project record's added decisions), validates it, stamps D4's record and commits both.
It refuses a beam whose run is not at Beam-Await, running or halted, a beam whose ready record stands, and a description that fails validation, naming the finding and `lym board upsert` for the entry's title or brief.
The writer moves from a producer to a function the verb calls; it stays in `landingshed`.
`lym beam describe` owns `summary.md` and writes it whole each run, so the orch never edits the file by hand; the orch's own text reaches it only through the entry's title and brief, set with `lym board upsert`.
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
  `mergeable_now` is true only for an open, non-draft pull request whose head is the beam's head and whose `mergeable_state` is `clean`, `has_hooks` or `unstable`; `dirty` is a conflict, `blocked` a review or required check, `behind` a base ahead of the head, and `unknown` is retried for up to one minute, the length of Finalize's merge retry window, and then reported as not yet computed; a closed or merged pull request is named as such.
  D3's refusal 6 calls the same function.
- `close`: closes the recorded pull request on GitHub, skipping the GitHub call for one already closed or merged, and drops it from D4.
  Where the resolver answers `on` or `off`, its GitHub comment names the operator's choice to land without a pull request.
  Elsewhere the comment names only the close.
  Where it answers `merge`, `close` reports that Finalize will still land through a pull request it finds or opens (D8); where it answers a required pull request, `close` reports that `lym beam ready` refuses until one is recorded again with `lym beam pr open <slug>` (D3).

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
  A merge GitHub refuses as pending (`pullrequest`'s RefusalPending) is retried within Finalize's existing one-minute merge retry window (`prMergeRetryWindow`).
  A refusal still pending past that window, every RefusalOther, such as a required check the push restarted or an approval it dismissed, and a RefusalMethod, a repository that disallows squash merges, halt naming GitHub's message, `lym beam pr status <slug>` and `lym loom resume`, never `lym batten cancel`.
  A recorded pull request GitHub reports closed halts naming the operator reopening it on GitHub and `lym loom resume`.
  One merged at the beam's head lands through the already-landed path.
  One merged at another head, the mismatch PR-Review halts on today, lands as with none recorded: Finalize pulls the parent first, so its merge-in carries GitHub's merge and the landing carries only the commits that merge lacked, an empty squash when it lacked none; the pull request is dropped from D4 after the parent-side landing, as always.
- None recorded in D4, and the run's scratch pull-request record holds an open pull request, one Finalize opened before a halt or one the old binary's PR-Review left: Finalize merges it as it merges a pull request it opened, without editing its text.
- None recorded, and a pull request is required (beam.yaml `required` or `merge`, or GitHub's rules): Finalize finds an open pull request from the beam branch, leaving its text as it is, or opens one from the description, and merges it, as `protected-parent-landing` built for `merge`.
  Finalize keeps a pull request it finds or opens in its scratch pull-request record, as today.
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
Bound: `lym loom goto` stays the operator's hand escape.
A goto back to Beam-Await while the ready record stands meets D11's entry rule: the entry finds the hold named Finalize with no land request, runs the accept's re-check itself, and is Done or refuses as the accept does.
The goto adds no way to take new work, since D2's guards start no member while the ready record stands.

### D11 — Beam-Await survives a loop exit

Beam-Await, Beam-Up and Beam-Down get the re-invoke interrupt policy, since each spawns no agent and is idempotent.
The driver's rules re-step a re-invoke stop (`internal/sheddriver/driverules.go`), and the re-invoke entries in loom's interrupt-policy table (`loomshed.InterruptPolicies`) make an interrupted pawl stop eligible under the arbiter's existing rule, so a loop exit at Beam-Await past the driver's repair cap is still resumed with no hand `lym loom resume`.

Beam-Await stamps the time of each completed wake in a never-tracked wake record in the run's scratch directory.
The driver reads a row's completed-wake stamp through a seam the pawl wiring tells, as D13's side effects hang on one, so its rule stays recipe-agnostic: the repair count for a stop key whose row supplies a stamp restarts at zero only when that stamp is at least ten minutes later than the `At` of the key's last restep settle record, so a beam waiting for days survives every exit.
Bound: a loop that dies within ten minutes of each re-step is re-stepped at most the repair cap's times and then reaches the tackler and the arbiter; a loop that lives longer is re-stepped at most the cap's times per ten minutes.

Beam-Await's entry reads the board status before it writes its own.
Finding the landing hold with no land request, a crash between the accept's removals and Done, it reads the tracked ready record for this slug.
If the record stands, the entry runs the accept's re-check itself, with no land request, and is Done or refuses as the accept does.
If it does not stand, the entry lifts the hold and starts the dobby whether or not the dobby-stop marker stands.
The dobby never writes the land request again here, since its land episode stays recorded (Constraints).
That entry check reads the hold by the Finalize name or a retired row name (D13), so a crash under the old binary is caught too.
Rationale: issue #57.

### D12 — `lym beam start` in landing

`lym beam start` on a beam its own pawl run holds, Finalize and Beam-Down included, brings back the beam's orch and leaves its dobby down, reporting that the dobby stays down while the landing runs; the refusal (`internal/orchcli/refusal.go`) stays for a beam held by anything else.
Bound: `lym beam start` never starts the dobby of a held beam; Beam-Down still takes the orch down.
Rationale: issue #47; the landing tolerates a live orch, and the operator needs it to follow a halted Finalize.

### D13 — In-flight pawl runs on a removed row

A recipe may declare retired rows, each mapped to the row a run on it moves to; pawl declares Parent-Align, Beam-Describe and PR-Review, each to Beam-Await.
Shed's step, run by a new-binary process and finding a status whose current row is retired, moves it once under the status lock: the status names the target row, paused with a reason naming the move and `lym loom resume`; the entry's board status names the target too, lifting the hold.
The driver's rule table hands a paused run back to the parent, never to the tackler or the arbiter, so the run stays paused for the orch.
For pawl the move also:

- ends a live PR-Review session of the run through `runproc`;
- carries an open pull request from the run's scratch record into D4's record, with the beam's head as its announce head, and removes the scratch record;
- removes the ready record, the land request, the dobby-stop marker and the reopen request;
- commits the carry into D4's record and the ready record's removal together in the beam worktree, through `fabricengine.CommitAnchoredPaths` over the landing directory;
- posts one notice to the beam's orch: the run was moved to Beam-Await and paused, the pull request stays open, run `lym beam start <slug>` and `lym loom resume`, a second resume when the move was made by a resume, then the new verbs.

A run at Finalize or past it is untouched; D8 lands through an open pull request the old PR-Review left in its scratch record.
Bound: only a status naming a row the recipe lists as retired moves; any other unknown row keeps today's refusal naming `goto`.
A driver process of the old binary still alive keeps running the old rows until it stops; the move happens at the first step a new-binary process makes.
Rationale: the row lookup refuses a status naming a row the recipe lacks today, and no row-migration mechanism exists; a recipe-declared map keeps it in shed's shared machinery.

### D14 — The beam orch's landing instructions

The orch role (`contracts/orchrole/orch-role.md`) carries a beam's landing workflow: describe, open, update and check the pull request when the operator says so, change the description only through the entry's title and brief, merge the parent in through `lym beam merge-parent`, act on the member-landing notices, and run `lym beam ready` only when the operator is satisfied, knowing it is final.
The `ready`, `pr` and `describe` verbs' help texts state the same, and the removed `reopen` text goes.

### Added after Discussion (parent, 2026-11-05): Issue #47's push-lock point drops

Decision: The board entry's point that a push lock whose holder died before a restart is taken as stale drops: a flock whose holder died is already free, so the `fabric.push.lock still held after 30s` seen in #47 was a live push left running from before the restart (the detached push child fabric spawns), doing legitimate work. lym does not stop such a push. How long a push holds its locks is the scope of the separate task `push-holds-remote-refs`, not of this task. The writer records the drop in Scope and states the researched cause in the open risk in place of `likely`. Settles review finding F7.

Rationale: The entry's premise is false for a flock; stopping a live push risks a half-written remote and widens scope into fabric, which another task already owns.

### Added after Discussion (parent, 2026-11-05): Beam-Await's entry re-checks a standing ready record itself (D10, D11)

Decision: D11: drop the sentence that starts the dobby when the dobby-stop marker stands and has the dobby write the land request again, and drop the claim that the dobby lands the beam again. Instead, Beam-Await's entry, on finding the hold named Finalize or a retired row with no land request, reads the tracked ready record for this slug. If the record stands, the entry runs the accept's re-check itself, with no land request, and then is Done or refuses as the accept does. If it does not stand, the entry lifts the hold and starts the dobby whether or not the marker stands. D10's Bound gives the same entry rule for a goto back to Beam-Await while the ready record stands. D10's and D11's acceptance criteria follow this. Constraints name the fact it rests on: the dobby's persisted land episode (dobbyengine State.Land) is cleared only on a non-passing reading, and the dobby-stop marker is already removed by the accept. Settles review finding F1.

Rationale: The review showed the dobby's persisted land episode stops it from rewriting the land request, so recovery must not depend on the dobby; the reviewer's fix keeps recovery inside the row that owns it.

## Constraints

- A flock is released when every descriptor of it is closed, and Go opens every file close-on-exec (Go `os` package docs; flock(2) man page), so a push lock whose holder process died is free already.
- GitHub's REST pull request object carries `mergeable` and `mergeable_state`, whose values are `clean`, `dirty`, `blocked`, `behind`, `unstable`, `has_hooks`, `unknown` and `draft`, mirroring the GraphQL `MergeStateStatus` enum (GitHub GraphQL schema, `MergeStateStatus`); GitHub computes mergeability asynchronously and reports unknown until done. D6 reads the REST fields through go-github, which `internal/pullrequest` already uses.
- `PATTERN-pull-request-mode`: the resolver stays the one decision; D8 adds the beam record as an input (PATTERN changes).
- `PATTERN-github-auth`: the `lym beam pr` verbs join the callers that open, reopen and close a pull request (PATTERN changes).
- `PATTERN-remote-refs-lock`: D3's parent fetch and D6's pushes go through `gitrepo` and fabric under the lock.
- `PATTERN-shed-shared-machinery`, `PATTERN-sole-parsers`, `PATTERN-shed-stop-chain`: D13's retired-row map is read by the recipe's sole parser and applied in shed's packages, and the run it leaves paused is handed back to the parent by the driver's rule table.
- The dobby keeps its land episode (`dobbyengine` `State.Land`) in its state file across restarts and clears it only on a reading that does not pass (`internal/dobbyengine/land.go`), so with the ready record standing and nothing working under the beam, neither it nor a restarted dobby writes the land request again; D10 and D11 rest on this.
- `classifyRefusal` in `internal/pullrequest` reads a pending required check and a dismissed review as RefusalOther; RefusalPending covers only a modified base and mergeability still being computed.
- `PATTERN-tracked-file-locks`, `PATTERN-durable-vs-ephemeral-state`, `PATTERN-tolerant-records`: D4's record is tracked with its lock under `.lym/locks/`; D7's announced head is never-tracked delivery state.

## Auto-mode assumptions

- "The hub's rules for beam pull requests" reads as the resolver over the parent's beam.yaml, GitHub's rules and the origin facts (D6, D8).
- The orch skips a pull request by not opening one or closing it (D8).

## Open risks

- Issue #47's push lock: the entry asks that "a push lock whose holder died before a restart is taken as stale", but such a lock is already free (Constraints); the wait #47 saw was a live holder, likely a push the restart left running. This record changes nothing there. Question for the parent: should lym stop a live push left from before a restart, and by which rule, or does this point drop?

## Acceptance criteria

- D1: the pawl recipe builds with the rows Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down, Beam-Await's `on_done` is Finalize, and no recipe row names BeamDescribe.
- D2: an accepted land request writes the entry status naming Finalize; `boardengine.RunHolds` reads a pawl run at Beam-Await as holding nothing and at Finalize as holding the beam; a beam dobby with the ready record standing starts no member run and posts the open-member notice once; a hand `lym batten run` of a member under a ready beam is refused naming `lym batten cancel <member> --remove`; a failing re-check writes Beam-Await back and restarts the dobby.
- D3: `lym beam ready` refuses, each with its way forward, a run not at Beam-Await, open work under the beam, a parent tip not an ancestor of the beam head (naming `lym beam merge-parent`), a parent with no upstream judged by its local tip alone, no description stamped for this beam, a required pull request with none recorded, a recorded pull request whose head differs from the beam's (naming `lym beam pr update`), and one not mergeable now or closed; a recorded pull request whose merge lacks no beam commit passes, the beam having moved past it only by merging the parent in included, and one whose merge lacks beam commits is refused naming them and `lym beam pr close`; it writes the ready record when none applies.
- D4: a landing record naming another slug reads as no record and is not removed; describe and `pr open` write and commit a record naming the beam; Finalize drops the pull request only after the parent-side landing.
- D5: `lym beam describe` writes and commits a description listing each beam commit with its pull request number or commit, merge time and summary, stamps the record, and refuses a run not at Beam-Await and a beam whose ready record stands; a halted run at Beam-Await is accepted.
- D6: `open` adopts an open pull request without editing it except a draft flag, creates one from the description otherwise, reopens a closed recorded one and answers `already` for an open recorded one; `update` sets title and body and the recorded head; `status` reports `mergeable_now` true only for an open non-draft pull request at the beam's head in `clean`, `has_hooks` or `unstable`, and names the cause otherwise, closed and merged included, and retries `unknown` for up to one minute before reporting it as not yet computed; `close` closes it, or skips GitHub when it is closed or merged, and drops it from the record, and reports on a `merge` parent that Finalize will still land through a pull request and on a required one that `lym beam ready` refuses until `lym beam pr open`; `open`, `update` and `close` refuse a run not at Beam-Await and a beam whose ready record stands, and `status` answers at Finalize.
- D7: with a pull request recorded, a member landing into the beam produces one notice to the beam's orch, a later pass with no new commit posts none, a member filed after the pull request opened is announced too, and a missing announced-head marker announces from the record's announce head; with none recorded, no notice.
- D8: a pawl Finalize with a recorded pull request squash-merges it on GitHub without editing its text, whatever `Config.Squash` says, and drops it from the record after the parent-side landing; a crash between the GitHub merge and the parent-side landing re-enters through the already-landed path; a refusal still pending past the one-minute retry window, a RefusalOther such as a pending required check or a dismissed review, and a RefusalMethod halt naming GitHub's message, `lym beam pr status` and `lym loom resume`; a recorded pull request merged at another head lands as with none recorded and is dropped from the record after the parent-side landing; with none in D4 and an open pull request in the scratch record, from the old PR-Review or a halted Finalize, it merges that one; with none recorded and a required or `merge` parent it finds or opens a pull request and merges it, halting on a GitHub refusal with `lym beam pr status`; with none recorded and `on` or `off` it lands by a local squash merge and push.
- D9: a pawl Finalize finding an open pull request merges it without a title or body edit, and a task run's Finalize behaves as today.
- D10: `lym beam reopen` is no longer a verb, no pawl row reroutes to Beam-Await, and a goto back to Beam-Await with the ready record standing re-checks and is Done with no land request.
- D11: Beam-Await, Beam-Up and Beam-Down carry the re-invoke entry in `loomshed.InterruptPolicies`, which makes an interrupted pawl stop at Beam-Await eligible for the arbiter with no change to `arbiterstop.Eligibility`; a loop crashing after each first wake is re-stepped at most the repair cap's times and then reaches the tackler; a loop exit after a completed wake stamped at least ten minutes past the last re-step is re-stepped again; a Beam-Await entry finding the hold named Finalize or a retired row with no land request runs the accept's re-check when the ready record stands, and is Done or refuses as the accept does, and otherwise lifts the hold and starts the dobby whether or not the dobby-stop marker stands.
- D12: `lym beam start` on a beam held by its own pawl run at Finalize starts its orch and not its dobby, and still refuses a beam held otherwise.
- D13: a pawl status naming Parent-Align, Beam-Describe or PR-Review is moved to Beam-Await, paused and handed back to the parent, with the board status naming Beam-Await, the scratch pull-request record carried into the landing record and removed, the ready record, land request, dobby-stop marker and reopen request removed, the carry and the ready record's removal committed together in the beam worktree, and one notice posted; a status naming a row neither present nor retired still refuses naming `goto`; a run at Finalize is untouched.
- Review: D14's orch role text describes the beam landing workflow and the finality of `lym beam ready`, and no text names `lym beam reopen`.

## PATTERN changes

- `PATTERN-pull-request-mode` — changes: Deciding if a landing uses a pull request: only landingshed's one resolver, from the receiving `beam.pull_request`, the entry's `pull_request` or a beam's pull-request record, the parent's GitHub rules, origin and task commits; no run setting carries it; a yes merges on GitHub.
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize, `lym beam pr` and pair teardown open, reopen, merge or close a pull request; `lym beam pr` never merges. (test)

## Notes for the plan writer

- D13's retired-row map is a recipe-level key read by the recipe parser; the pawl side effects (session end, record carry, file removals, notice) hang on a seam the pawl wiring tells, so shed stays recipe-agnostic.
- D7 fits beside the dobby's existing landing notice (`internal/dobbyengine/landnotice.go`), which posts through the same nearest-orch path; D2's start hold fits the dobby's start step beside its ready read (`internal/dobbyengine/land.go`).

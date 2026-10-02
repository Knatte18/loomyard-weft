# Decision record — run-parentage

## Goal

A run in a task pair knows its parent from the worktree the pair was created from, not from the caller's `LYX_STRAND_NAME`, so `lyx batten run <slug>` from a plain shell in the prime still reaches the prime's orch session.
Batten waits through a halted inner run instead of exiting `failed`, so the pair keeps a watcher that lands and tears it down.
Parent-review reviews every rewrite after a reject, up to a cap, and escalates at the cap instead of letting the rewrite through unseen.
Fixes #341 and #338.

## Scope

In:

- Recording the parent worktree when `lyx fabric add` (and batten's Worktree-Create, which goes through it) creates a pair.
- Resolving that worktree to its orch session's agent name wherever a parent name is read: reed's `LYX_PARENT` export and the parent-review gate's reviewer.
- Retiring the caller-env parent writes in `lyx batten run`'s arm, batten's Seed-Child copy and `lyx loom start`'s bootstrap.
- Batten's Run-Shed row waiting through `paused`, `blocked` and `failed` children.
- Multi-round parent-review with a durable reject cap that escalates.
- Docs in the same commit: the Agent Name Invariant in `CONSTRAINTS.md`, the `shedrun`, `battenshed`, `parentreview` and `fabricengine` package docs, the loom and batten recipe comments, `lyx loom review`'s help text, and deleting CLAUDE.md's TEMPORARY items for #341 and #338.

Out:

- A mailbox or queue for messages to an absent orch (the board's `creel` task, deferred by the operator).
- Any Go process typing into the orch pane, including batten.
- The parent-review timeout policy: a request with no verdict after `parent_review_wait_min` still passes with a Warn.
- Walking a parent chain upward to find an orch in some ancestor worktree.
- The other items folded into `pair-lifecycle`, except where Decision 6 resolves its `batten-resume-sees-stale-blocked` item as a side effect.

## Decisions

### 1. The pair's origin record holds the parent worktree

Decision: `fabricengine`'s origin record gains a write-once `parent_worktree` field holding the `WorktreeName` of the worktree `fabric add` ran from, written in the same step and commit as `parent_branch`.
Batten's Worktree-Create gets the prime this way, since batten runs from the prime only.
The legacy-worktree origin write in `lyx loom start`'s bootstrap (a pair created before the origin record existed) leaves the field empty, since its creator is unknown.

Rationale: the origin record is already the pair's write-once provenance, written once at creation and committed to the weft, so "the worktree it is seeded from" lands beside "the branch it is forked from".
Recording it at creation covers both seeding routes, batten's Seed-Child and a hand-run `lyx loom start`, without either needing to know who called it.

### 2. The seed stops recording a parent

Decision: no code path writes `shedrun.Seed.Parent` any more; `lyx batten run`'s arm, Seed-Child and `loomSeedFor` drop it.
The field stays in the struct so a seed written before this change still decodes under `DisallowUnknownFields`, and it is read only by Decision 3's legacy fallback.

Rationale: one record of parentage, in fabric, with no environment read anywhere on the write side; that removes #341's cause rather than adding a second source.

### 3. Parent names resolve at use, never stored

Decision: one resolver, owned outside `parentreview` and `reedengine` (both told-geometry), maps a pair's location to its parent agent name:

- origin record with `parent_worktree` set: the orch role's full name in that worktree, formed through `internal/agentname` from the hub shortname, the parent worktree's slug (none for the prime) and `agentname.RoleOrch`;
- no `parent_worktree`, legacy seed `parent` non-empty: that legacy name verbatim;
- otherwise (no origin record, as in a standalone run or the prime itself): no parent.

`hubgeom` uses it for reed's `ParentName`, so every strand in the pair gets `LYX_PARENT`; the parent-review wiring uses it for the reviewer and the delivery prompt.
The resolver checks no liveness: the formed name is exported even when no orch strand is live, so an orch started later is reached by the same name.

Rationale: the name is derivable from the worktree and the hub shortname, both stable, so storing it would only add a copy that can drift.
The legacy branch keeps runs in flight at deploy time, which carry `ly:orch` from the #341 workaround, parented with no hand edit.
The fallback is bounded to seeds that already exist: Decision 2 means no new seed carries a parent.

### 4. With no live orch in the parent worktree, nothing is queued

Decision: an upward SendMessage to an absent session stays what the driver prompt already says, a failed send treated as no send; the drive report and the parent-review round files stay on disk as the durable record.
The parent-review gate alone holds: while no live strand in the parent worktree's reed session carries the reviewer name, the gate stays pending without sending the writer a delivery prompt and without counting one against `maxGatePrompts`, and logs one Info line per round that it is holding.
Once such a strand is live, the gate prompts as today.
The hold is bounded by the existing `parent_review_wait_min`; past it, the round expires and passes with the existing Warn.
A legacy-name reviewer (Decision 3's fallback) has no known worktree and is treated as live.
The liveness check reaches `parentreview` through a told seam on `GateConfig`.
A failed liveness read counts as not live: the gate holds and logs a Warn naming the error, never returns it as a gate error.

Bound: the hold delays the delivery prompt, never skips the review short of that wait bound; a reviewer that never becomes live, as for a nested pair (Decision 3 resolves its reviewer to an orch name in a pair, where `lyx orch` never runs), always ends in that timed pass.

Rationale: the gate's three prompts at one-minute throttle would otherwise burn out during an orch relaunch and leave the request to time out unreviewed, and holding the prompt until the orch appears costs one liveness read per evaluation.

### 5. Parent-review loops, with a durable reject cap that escalates

Decision: after a consumed reject, the writer's next arrival that passes the `discussion` gate opens a new round (`round-<N+1>`) with a fresh request, brief and delivery prompt; the parent reviews every rewrite.
The entry stops being `pass_on_cap`; its `attempts: 3` is told to the gate as the cap of 3 rejected rounds.
The round store decides the cap: the gate counts rounds whose verdict is a reject, so an attach, a driver restart or a resumed run never resets the count.
A reject below the cap is an ordinary failure, which the engine re-prompts with the review path.
The reject that brings the count to the cap returns a new terminal failure on `GateResult`, which the engine finalizes as failed without a re-prompt whatever its in-memory count.
The in-memory count never exceeds the store count, so the engine's own budget never fails the entry first.
At the cap, Discussion-Write ends gate-failed Stuck, its discussion is committed as today, and the run halts `blocked` with a reason naming the slug, the round count, the latest review file and the way out.

Discussion-Write's fresh-spawn preparation keeps the latest round when its verdict is a reject or a superseding approve, and otherwise opens the next round as today.
The gate reads the latest round before opening a request: a superseding approve passes, the cap's reject fails at once without a new request, an unconsumed reject below the cap is consumed and re-prompts with its review, and a consumed one opens the next round.

The way out is `lyx loom review approve <slug>`, which on a run halted at the cap replaces the cap's reject with an approve marked superseding, keeping its review file; the next `lyx loom start` re-spawns the writer, whose first arrival passes the entry.
Without that approval, a resume's first arrival fails the entry at once and the run halts again.

The engine's refusal of `Pending` and `Final` from a must-pass entry is lifted only for an entry whose new `GateEntry` field allows it; only the parent-review wiring sets the field, never an entry name or a recipe key.

Bound: the cap turns `lyx loom review reject`, which checks no caller identity, from an advisory verb into one that can halt a run, but only after three rejected rounds, and only into `blocked`, which the parent or operator lifts with one approve.
Approve's supersede path applies only to a run halted at the cap: its status is `blocked`, the store holds cap rejected rounds, and the latest round's verdict is the cap's reject.
Otherwise approve refuses to supersede, and it never rewrites an earlier round or an open round's pending state.
Approve, like reject, checks no caller identity, so that halted state is the only gate on the supersede path.
The lifted engine guard lets nothing through: the entry may hold the run and set `Final`, but passes only on an approve, an expired request (at its wait bound or at a finalize, as today), no reviewer or no open request.
`Final` fails on an unconsumed reject and counts it toward the cap as the gate does, so a lost session or a deadline after a reject never lets the rejected discussion through.
`lyx loom review`'s help text states the reject and approve bounds.

Rationale: three rounds give the writer two rewrites to converge on the parent's findings while keeping the parent's attention bounded; escalating at the cap is the board's fix direction, and an approval override is the parent's own verdict, so no new verb is needed.

### 6. Batten waits through a halted child

Decision: Run-Shed's `paused`, `blocked` and `failed` arms become budget-exempt waits, like the `awaiting` arm, instead of hard errors.
Each wait's stuck reason names the child's state, error and current producer, and the resume command (`lyx loom start` in the task worktree).
Batten never resumes the child itself.
When the child is `running` again, the running arm takes over; `done` reaches teardown as today.
Batten logs one Warn per halt episode, keyed by the child's history length so a re-halt after a resume logs again, and stays quiet on the polls between.
The operator stops the wait with `lyx batten pause <slug>`.

Bound: the wait has no time limit and spends no bounce budget, so a child nobody resumes keeps batten polling every `poll_interval_s`; `lyx batten pause` and `lyx batten status` are the operator's handles on it.
A `failed` child is included because `lyx loom start` resumes `failed` like `blocked` and `paused`, and the watcher gap is the same.

Rationale: #338's cause is the hard-error arm.
Making the wait budget-exempt also covers `pair-lifecycle`'s `batten-resume-sees-stale-blocked` case, a `blocked` status read in the seconds after `lyx loom start`.

### 7. The child's driver is the one that tells the parent

Decision: batten sends no message of its own on a halt.
The child's llm driver already messages `LYX_PARENT` with the run-id and report path at every stop other than done, which covers `paused`, `blocked` and `failed`, and with Decisions 1 and 3 that variable is set whoever ran `lyx batten run`.
Batten's tell is its Warn and stuck reason, which `lyx batten status` shows.

Rationale: batten is a Go process with no SendMessage, and the driver's message already carries what the parent needs to act.

## Constraints

- The Agent Name Invariant in `CONSTRAINTS.md` gains the parentage rule in the same commit: a run's parent is the worktree its pair was created from, recorded once in fabric's origin record; its agent name is resolved at use as the orch role's name in that worktree and is never stored in a seed or read from the caller's environment.
- `internal/agentname` stays the sole former of the parent's name; the resolver forms it through `agentname`, never by string concatenation.
- `internal/fabricengine` stays the sole reader and writer of the origin record.
- `parentreview` and `battenshed` stay told-geometry: the liveness check and every path arrive through their configs.
- The Fabric Vocabulary Invariant holds: `battenshed` text says "the task worktree" and "the pair", never a fabric side.
- Every agent stays an interactive tmux session driven through reed; no Go process sends into the orch session except the orch watcher, and batten sends into no session.
- Docs land in the same commit (see Scope); `manifest/roadmap.md` is untouched.

## Auto-mode assumptions

- The open question "queue until an orch appears, or log only" was settled without the operator as Decision 4: log only, with the parent-review gate holding its prompt within its existing wait bound.
- The reject cap of 3 was picked without the operator.
- `Final` failing on an unconsumed reject (Decision 5) replaces today's Warn-and-pass, picked without the operator.
- Including `failed` in batten's waits goes one state past the board's "paused and blocked".
- Decision 7 departs from the board's "tells the parent once" wording: the child's driver's existing stop message is the tell, and batten adds none.
- The parent-review timeout still passes with a Warn, as today.

## Open risks

- A nested pair (created from another pair) resolves to an orch name in that pair, where `lyx orch` never runs, so its upward messages go nowhere; the same as a plain-shell run today, but now by design.
- A child with `child_driver: go` has no driver message, so its halts reach the parent only through batten's status.
- `pair-lifecycle` still lists `batten-resume-sees-stale-blocked`; once this lands the hub should drop that item from the board task.
- Legacy fallback (Decision 3) stays in code until no seed with a recorded parent remains in flight; it is small, but nothing removes it automatically.

## Acceptance criteria

- `lyx batten run <slug>` from the prime with `LYX_STRAND_NAME` unset creates a pair whose origin record names the prime; every strand in the pair sees `LYX_PARENT` equal to the prime's orch name; and the parent-review request's reviewer is that name.
- A pair created by `lyx fabric add` from another pair records that pair as `parent_worktree`.
- Neither `lyx batten run`'s arm, Seed-Child nor `lyx loom start` writes a seed `parent`; a pre-existing seed carrying one still decodes.
- Resolver table: prime parent → `<shortname>:orch`; pair parent → `<shortname>:<slug>:orch`; no origin record → none; origin without `parent_worktree` and a legacy seed name → that name; both absent → none.
- Parent-review gate: a reject followed by a passing rewrite opens round 2 and prompts delivery; an approve on round 2 passes; the third reject fails the entry and the run halts `blocked` with the reason above; the reject count survives a fresh gate built over the same store, and an attach between rejects does not move the cap; `lyx loom review approve` on a run halted at the cap supersedes the latest reject and the re-spawned writer's first arrival passes; a timeout still passes with a Warn.
- Parent-review refusals: approve's supersede refuses on a run that is not `blocked` and on a latest round that is not the cap's reject; a resume without approval fails the entry at once, opening no request; a lost session or a deadline after an unconsumed reject fails the entry, and the re-spawned writer is re-prompted with that review.
- Parent-review hold: with the reviewer not live, the gate stays pending, sends no prompt and spends no prompt count; once live, it prompts; a legacy-name reviewer is treated as live; a failing liveness read holds with a Warn rather than erroring.
- Batten Run-Shed: a `paused`, `blocked` or `failed` child gives a budget-exempt Stuck naming the state and the resume command, not an error; a child going back to `running` and then `done` reaches teardown; one Warn per halt episode across many polls, and another after a resume and re-halt.
- End to end: pausing and resuming an inner run leaves its batten watcher alive and the pair is torn down after done.
- CLAUDE.md's TEMPORARY items for #341 and #338 are gone, and the Agent Name Invariant states the parentage rule.

## Notes for the plan writer

- Parent write sites today: `internal/battencli/arm.go` (env read), `internal/battencli/wire.go` (Seed-Child copy), `internal/loomcli/sharedbootstrap.go` (`loomSeedFor`'s caller); read sites: `internal/hubgeom` (`reedGeometry`) and `internal/loomcli/wiring.go` (`newParentReviewConfig`).
- The origin record is `internal/fabricengine/origin.go`, written in `add.go` and in the bootstrap's legacy branch.
- `shuttleengine`'s `evaluateGate` refuses `Pending` and `Final` from a non-`pass_on_cap` entry today; `handleGatedBoundary` finalizes a failing entry only once its in-memory count reaches `Attempts`, and `GateEntry.PassOnCap`'s doc notes that count resets on attach, which is why the cap's reject needs its own terminal result.
- Discussion-Write's fresh-spawn preparation calling `Store.BeginRound` is in `internal/shedrecipe/entries_discussionwrite.go`.
- Reed state is per worktree session; the liveness seam needs a reed engine built over the parent worktree's geometry.
- `battenshed/innerrun.go`'s `callAwaiting` and `exemptWait` are the model for the new halted-wait arm.

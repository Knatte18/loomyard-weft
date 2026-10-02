# Parent review as a step in Discussion-Write's gate

## Goal

Port millhouse's `orch-review`/`orch-wait` into Discussion-Write's own producer gate.
When the writer reports done and the mechanical `discussion` validator passes, a reviewer (the run's parent, recorded in its seed) reviews the discussion pair once.
A reject re-prompts the still-live writer with the review, so the writer fixes it with its own context.
The next arrival re-runs the mechanical validator and then lets the run through to the Discussion-Review segment as today.
No new recipe row and no separate fixer.

Alongside it, Discussion-Write and the reviewer both read the board, so a design fits planned work and neither duplicates nor contradicts it.

## Scope

In:

- A third gate-result state, **pending**, in `shuttleengine`'s gate contract, and the wait loop's handling of it.
- A new gate name `parent-review` in `shedrecipe`'s gate vocabulary, and the Discussion-Write row's `gates` list gaining it after `discussion`.
- The review exchange on disk: a per-round request directory holding the request, the delivery record, the verdict and the review file, committed to weft with the discussion.
- Delivery of the notice by the live writer through Claude Code's `SendMessage` to the parent's full name.
- A `lyx loom review` verb subtree: `notify`, `delivered`, `approve`, `reject`.
- A reviewer brief stencil the request renders, pointing at the review-output format and the board.
- Status: `lyx loom status` and `lyx batten status` show a run waiting on its reviewer and who it is.
- A loom config key for the review wait bound.
- Discussion-Write's stencil: read the board for design context, and handle a parent-review re-prompt.
- The orch start and resume stencils: review requests are handled through a one-shot fork.
- Docs, refusal-spec rows and tests for all of the above.

Out:

- Live questions from the writer to the parent (the separate board note `discussion-advisor`).
- A parent review in Plan-Write's gate or in Plan-Review round 1; `parent-review` is wired to Discussion-Write only.
- Reviewers other than the parent; the request records a reviewer name, and nothing else configures one.
- Any change to the Discussion-Review (Bouncer/Burler) segment.

## Decisions

### D1 — The review is a pass-on-cap gate entry on Discussion-Write

Decision: Discussion-Write's `gates` list becomes `discussion` (`attempts: 3`) followed by `parent-review` (`attempts: 1`, `pass_on_cap: true`).
`attempts: 0` turns the review off.
Discussion-Burler's gate list is unchanged.

Rationale: the gate list already evaluates entries in order and stops at the first failure, so the mechanical validator always runs before the review and re-runs on every later arrival.
With `attempts: 1` and `pass_on_cap`, a reject re-prompts once and the next arrival is let through, which is `orch-review`'s run-once behaviour; independent reviews still follow in the Discussion-Review segment.

### D2 — Gate results gain a pending state

Decision: `GateResult` gains a pending state, and `GateEntryState` gains `waiting`.
A pending result may carry one line of text for the loop to send the writer (D5's delivery prompt); the closure never sends anything itself.
In the wait loop, a pending entry sends no re-prompt and changes no count.
While an entry is pending, the loop keeps polling and re-evaluates the gate list on a poll tick without a new arrival, but only while the writer is at a turn boundary: its last arrival was Done and the loop has sent it nothing since.
Every gate send, whether a pending result's text or a reject's re-prompt, happens only at that boundary, so no gate send lands inside a writer turn.
A verdict submitted while the writer is mid-turn is therefore read at the turn's Done arrival.
The Done arrival that ends a delivery-prompt turn is an ordinary arrival: the gate list runs from `discussion`, and the pending entry is reached again.
A failed send of a pending result's text is logged at Warn and leaves the entry pending; it does not end the loop the way a failed re-prompt send does.
Liveness checks and the run deadline keep running throughout.
`finalize` never waits: an entry still pending at the final verdict is reported `waiting` and counts toward `Passed` like a let-through entry.
`finalize` evaluates a pending-capable entry without its opening and consuming transitions: it opens no request and consumes no verdict (D4).
Bound: only a `PassOnCap` entry may return pending.
A pending result from any other entry is a gate error, so pending can never let a must-pass gate through.
`shedrecipe` rejects `parent-review` without `pass_on_cap: true` as a construction error naming the row.

Rationale: a closure that blocks for minutes would stall the wait loop's liveness and deadline checks.
A pending state keeps the existing loop in charge of time and lets a driver restart re-attach to a gate that is still waiting.

### D3 — The review exchange is files in a per-round request directory

Decision: each review round has one directory, holding:

- the request: run slug, round number, reviewer full name, opened-at time, the discussion pair's paths, and the rendered reviewer brief;
- the delivery record: delivered-at time or the last failure reason, the number of gate delivery prompts carried and the time of the last one, and the number of `notify` prompts waiting;
- the verdict: approve or reject, recorded-at time, and whether the gate has consumed it;
- the review file, copied in by the verdict verb.

The directory sits beside the Discussion-Review run directory under the run root, never under `_lyx/discussion/`, which `discussionparser` owns.
It is committed to weft with the discussion pair at the discussion commit seam.
A fresh Discussion-Write run (a start, not an attach) opens the next round number; an attach continues the latest round.
Opening a round marks any earlier round without a verdict superseded; the verbs (D6), the closure and status (D9) act on the latest round only.

Rationale: files survive a driver restart, so `AttachGated` finds the open request and keeps waiting rather than asking twice, the prompt count and throttle continue where they stopped, and a `notify` run in another process reaches the gate.
The committed review is the durable record of what the parent said.

### D4 — The `parent-review` closure

Decision: on each call the closure reads the current round's directory and returns:

- no parent recorded in the run's seed: passed, logged once at Info;
- no request yet: opens the request, returns pending carrying the delivery prompt (D5);
- request open, no verdict, wait bound not reached: returns pending, carrying the delivery prompt again only when D5's re-send rule or a waiting `notify` asks for it, and updating the delivery record's counts when it does;
- wait bound reached with no verdict: passed, with a Warn naming the reviewer and the request path, and the request marked expired;
- verdict approve: passed;
- verdict reject, not yet consumed: marks the verdict consumed, returns failed with findings naming the copied review file;
- verdict reject, already consumed: passed.

The wait bound is measured from the request's opened-at time, so a restart does not reset it.
Evaluated by `finalize` (D2), the closure opens no request, carries no prompt and consumes no verdict.
A request still open without a verdict is marked expired and the entry reported `waiting`.
A recorded reject that was never consumed stays unconsumed in the committed directory, and `finalize` logs a Warn naming its review file.
Bound on the consumed marker: if the driver dies, or the re-prompt send fails, between the closure marking a reject consumed and the writer receiving the re-prompt, the writer never sees that review, and the next evaluation passes; the independent Discussion-Review rounds still run.

Rationale: the closure is idempotent over the on-disk state, which is what the gate-list commit asks of a request that must not repeat across an attach.

### D5 — The live writer delivers the notice through SendMessage

Decision: when the closure opens a request, the wait loop sends the still-live writer one delivery prompt, carried by the pending result (D2), instead of typing into the reviewer's pane.
The prompt is a stencil, rendered with the run slug, the request path and the reviewer's full name from the seed; it tells the writer to send one message through Claude Code's `SendMessage` addressed to that full name, carrying the slug, the request path and an instruction to review with a one-shot fork, then to end its turn without changing the discussion.
`$LYX_PARENT` holds the same name, since reed exports the seed's parent as it, but the prompt names the address itself.
The delivery prompt is not a re-prompt: it changes no gate count, and the entry stays pending.
The writer records delivery by running `lyx loom review delivered [<slug>]` after the send, which stamps the delivery record; a send that fails is recorded with `--failed <reason>`.
If no delivered-at time is recorded a minute after the last prompt, the closure carries the prompt again, up to three prompts per round; a `--failed` record is not a delivery and does not stop the re-send.
Each time the closure carries a prompt it increments the delivery record's prompt count and stamps the last-prompt time, so a prompt counts when the gate hands it to the loop, whether or not the loop's send succeeds.
A failed send therefore spends one of the three, and its Warn (D2) names `notify` as the way forward.
After the third prompt the gate logs one Warn and stops re-sending.
Each `lyx loom review notify` adds one to the delivery record's waiting-notify count; at the next turn boundary the closure carries one prompt per waiting notify and decrements the count, outside the cap and the throttle.
A failed or absent delivery is logged at Warn and never refuses or stops the run.
The SendMessage tool and the `$LYX_PARENT` variable are Claude- and session-specific, so the delivery prompt stencil and any provider wording live behind the provider seam; `shuttleengine` and `reedengine` gain no Claude specifics.

Bound: delivery never types into the reviewer's pane, so it cannot land inside a turn, a draft or an orch handoff.
The parent receives at most three notices per round from the gate, plus one per `notify` run.
The counts are on disk (D3), so the bound holds across driver restarts.

Rationale: the hub is hand-started today and is not a reed strand, so reed typing would never reach it and every run would wait out the full bound;
SendMessage to `$LYX_PARENT` already carries every driver's escalation notice to the hub since `agent-naming` landed, and the operator chose Claude Code's own messaging over a lyx mailbox for now.

### D6 — `lyx loom review` verbs

Decision: a new `review` subtree under `lyx loom`:

- `lyx loom review notify [<slug>]` adds one waiting notify to the open request's delivery record, and the gate carries one delivery prompt for it at the writer's next turn boundary, bypassing the once-a-minute throttle and the three-prompt cap (D5).
  It refuses when no request is open, naming that only Discussion-Write's gate opens one, and when the open request already has a verdict or has expired.
- `lyx loom review delivered [<slug>] [--failed <reason>]` stamps the open request's delivery record; the writer runs it after its SendMessage.
  It refuses when no request is open, and when the open request has expired.
- `lyx loom review approve [<slug>] [--review <file>]` records an approve verdict; the optional review file is copied in as a record only.
- `lyx loom review reject [<slug>] <review-file>` records a reject verdict and copies the review file in.
  It refuses a missing or empty review file.

`<slug>` is optional inside a task worktree and required from the hub's prime worktree, where the verb resolves the task worktree through the hub layout.
`approve` and `reject` refuse when no request is open, and when the open request already has a verdict or has expired.
Every refusal names its way forward and gets a row in `contracts/specs/refusal-spec.md`'s loom section with a reaching test.
The verbs do not check who runs them; any session or the operator can submit.

Bound: an unchecked verb reaches no further than this one advisory review.
An `approve`, including the writer approving its own discussion, skips at most the parent review, which is pass-on-cap and which wait-bound expiry skips anyway.
A `delivered` stamped without a send stops the re-sends, so the parent may never hear of the request; the run then waits out `parent_review_wait_min` and passes, and status shows the wait throughout (D9).
A `reject` spends at most the single pass-on-cap re-prompt.
A `notify` sends one delivery prompt per invocation and changes no gate count.
The independent Discussion-Review rounds run after every outcome, and the committed request directory keeps the verdict and delivery record visible to them and to the operator.

Rationale: the reviewer (the hub's fork) runs in the prime, not in the task worktree, so the verbs must address a run by slug; the `approve`/`reject` split mirrors the PR-Gate idiom.

### D7 — The reviewer brief

Decision: a new stencil, rendered into the request, tells the reviewer to:

- read the decision record and the support log at the request's paths;
- read `lyx board list`, and `lyx board get` any entry that touches the task, and judge whether the design fits planned work without duplicating or contradicting it;
- write the review in the review-output format (BLOCKING/NIT; class design/scope/decision/consistency), by pointer to the stencil that defines it, never restating it;
- submit with `lyx loom review approve` or `lyx loom review reject`, with the exact command lines for this run.

A review with any finding is a reject; an approve carries no finding the writer must act on.

Rationale: the Producer Pointer-Rule Invariant forbids duplicating the format contract, and a self-contained brief lets a fork review with no prior context.

### D8 — The writer addresses every finding, by fix or by answer

Decision: the reject re-prompt's findings tell the writer to address every finding, BLOCKING and NIT alike, either by fixing it or by answering it.
Answers go in the support log's `## Review rounds` section, as one parent-review entry listing each finding with its disposition (fixed, or answered with the reason).
The Discussion-Write stencil gains a short paragraph describing this re-prompt and the delivery prompt (D5).
Its "What you may write" section gains two carve-outs: running `lyx loom review delivered` when a delivery prompt asks, and appending the one parent-review entry to `## Review rounds` when a parent-review re-prompt asks.
Every other `## Review rounds` entry stays the Discussion-review gate's.
Discussion-Review's independent rounds read that entry with the rest of the support log.

Rationale: the writer keeps its context and may disagree, and the support log is where the review history already lives and what the next reviewers already read.

### D9 — Status shows the wait

Decision: while a round has an open request without a verdict, `lyx loom status` reports the run as waiting on its reviewer, naming the reviewer's full name, the open time and whether the notice was delivered, in place of a plain `running`.
Only the latest round is reported; a superseded round (D3) is not.
The request location reaches `shedverbs` as a told value through its `Spec`, so `shedverbs` stays path-free.
`battenshed`'s InnerRun row carries the same information into its own wait message, so `lyx batten status` in the prime shows which child waits on which reviewer.

Rationale: a wait of minutes must be visible from both sides, and the hub sees its children through batten.

### D10 — Wait bound in loom config

Decision: `loom.yaml` gains `parent_review_wait_min`, default 60.
`discussion_timeout_min` still bounds the whole Discussion-Write run, review wait included; if it expires first, the deadline path finalizes and the pending entry reports `waiting` (D2).

Rationale: an absent or busy parent must not stall a task indefinitely, and 60 minutes covers a busy hub plus a fork review well inside the 480-minute discussion deadline.

### D11 — Discussion-Write reads the board

Decision: Discussion-Write's stencil tells the writer to run `lyx board list` and `lyx board get` any entry that touches its task before interviewing, and to treat planned work as design context.

Rationale: the board shows what is planned ahead; `lyx board notes` is an alias over the same store, so `list` alone covers both.

### D12 — The hub forks to review

Decision: the orch start and resume stencils gain one paragraph: a review-request notice is handled by a one-shot fork that reads the request, reviews and submits, so the hub's own context grows by the notice and the fork's summary per round.
A repeat notice for a request the hub has already forked for starts no second fork.

Rationale: that is the context-cost property `orch-review` had.

## Constraints

- **Shuttle Provider-Seam Invariant:** the delivery prompt's SendMessage wording stays behind the provider seam; neither `shuttleengine` nor `reedengine` references Claude specifics.
- **Agent Name Invariant:** the reviewer is addressed by the full name the seed records; nothing parses names outside `internal/agentname`.
- **Told-Geometry Invariant:** the closure, delivery and status hook receive every path as told; no engine imports `lyxcwd`.
- **Shed Verb-Set Invariant:** the waiting-on-reviewer status is a told hook in `shedverbs`'s `Spec`, never a reimplemented `status` body.
- **Transient Stop Invariant:** pending is not a verdict and never carries the transient mark.
- **Refusal Way-Forward Invariant:** every new loom refusal names its way forward and lands with its refusal-spec row and reaching test in the same commit.
- **Fabric Git Invariant:** the request directory is committed by Go at the discussion commit seam through `fabricengine.ScopedPathspec`, never by an agent.
- **Discussionparser Sole-Parser Invariant:** the request directory is not under `_lyx/discussion/`.
- **Gate Self-Check Parity Invariant:** `parent-review` is not a mechanical gate and gets no self-check verb; the invariant's list stays as is.
- **Stencil Ownership Invariant:** the reviewer brief is a stencil read at call time.
- **Live-Substrate Spawn Observability:** delivery starts no process; the delivery prompt goes through shuttle's existing re-prompt send path.
- **CLI/Cobra Invariant:** every new command has a non-empty `Short`, JSON errors, and a `clihelp.ShouldAbort` check; help-tree tests cover the new subtree.
- **Documentation:** `contracts/specs/shed-recipe-spec.md`'s gates paragraph (new name, pending, the `pass_on_cap` requirement), `contracts/specs/loom-status-spec.md`, `docs/overview.md`'s loom row, and the touched package docs land in the same commit.

## Auto-mode assumptions

- The parent is normally the hub, hand-started or hosted by `lyx orch start`; either receives the notice through SendMessage.
- 60 minutes is a sensible default wait bound.
- One round per Discussion-Write run is enough; the independent segment reviews carry any further iteration.
- An approve carries no actionable finding; a review with findings is submitted as a reject.

## Open risks

- **Writer compliance.** Delivery depends on the writer sending the message and stamping it; a writer that does neither leaves the request undelivered, the gate re-sends the delivery prompt once a minute up to the D5 cap, and status shows it undelivered.
- **SendMessage is a Claude Code internal.** A change to it breaks delivery; the request stays open and visible in status until the bound.
- **Message during a handoff.** A notice reaching an orch-hosted hub mid-cycle arrives in its next turn; the request stays open, so a lost notice is re-sent with `notify`.
- **Lost reject on a crash.** The consumed-marker window in D4 can drop one review.

## Acceptance criteria

- A Discussion-Write run whose seed has a parent opens a request after the `discussion` gate passes, sends the writer one delivery prompt naming the parent's full name, records the delivery once the writer runs `lyx loom review delivered`, and shows as waiting on that reviewer in `lyx loom status` and in the prime's `lyx batten status`.
- `lyx loom review approve <slug>` from the prime lets the run through to Discussion-Bouncer with no re-prompt.
- `lyx loom review reject <slug> <file>` re-prompts the live writer once with findings naming the copied review file.
  The next arrival re-runs `discussion`; a passing arrival is let through without a second review, and a failing one re-prompts on the `discussion` budget as today.
- A run with no parent in its seed, or with `attempts: 0`, passes the review entry without opening a request.
- A request with no verdict past `parent_review_wait_min` passes with a Warn, the request is marked expired, and later verdict verbs refuse with a way forward.
- A driver restart while waiting re-attaches to the same round: no second request, no second notice once delivered, and the wait bound continues from the original open time.
- A driver restart after two undelivered gate prompts sends at most one more gate prompt in that round, no sooner than a minute after the second; a `notify` run while the driver is down is carried at the first turn boundary after the restart.
- A fresh Discussion-Write start while an earlier round is open without a verdict marks that round superseded; the verbs and status act on the new round.
- An attach after a consumed reject passes the review entry.
- A fresh Discussion-Write start after a completed round opens the next round number.
- An undelivered request re-sends the delivery prompt at most once a minute and at most three times per round, and changes no gate count; `delivered --failed` records the reason with a Warn and leaves the run waiting and re-sending within the cap; `notify` sends one prompt at the next turn boundary, past the cap.
- A reject submitted while the writer is mid-turn is read at that turn's Done arrival and re-prompts once; no gate send happens while the writer is mid-turn.
- A failed delivery-prompt send logs a Warn naming `notify`, leaves the entry pending, and counts toward the three-prompt cap.
- No path of delivery types into the reviewer's pane.
- A live smoke with a hand-started parent session receives the notice through SendMessage.
- A pending result from a non-`PassOnCap` entry is a gate error; `shedrecipe` rejects `parent-review` without `pass_on_cap: true`.
- The wait loop runs liveness and the run deadline while an entry is pending; a deadline expiry reports the entry `waiting` and finalizes.
  The finalize-time evaluation opens no request and consumes no recorded reject; it marks an open request expired and logs a Warn naming an unconsumed reject's review file.
- Each new refusal of the `review` verbs (`notify`, `delivered`, `approve`, `reject`) has a refusal-spec row and a test reaching it: no open request, verdict already recorded (`notify`, `approve`, `reject`), request expired (all four), missing or empty review file, slug required from the prime, unknown slug.
- The request directory is in the discussion commit.
- The Discussion-Write stencil tells the writer to read `lyx board list` and how to address a parent-review re-prompt, and its write boundary permits `lyx loom review delivered` and the one parent-review entry; the reviewer brief points at the review-output format rather than restating it.
- Unit tests cover the closure's state table with a fake clock and a fake delivery seam, the wait loop's pending handling and turn-boundary sends, and recipe resolution.
  An integration test drives a gated run through request, reject, fix and let-through with a fake reviewer submitting through the verbs.

## Notes for the plan writer

- The gate list landed in `b7c478620`; `internal/shuttleengine/gate.go` and the gated branch of `wait.go`'s loop are where pending goes.
- `internal/shedrecipe/entries_gate.go`'s `resolveGateClosure` is the gate vocabulary; the run's parent lives in the seed (`shedrun.Seed.Parent`) and reaches `Env` from there.
- Drivers already notify the parent through SendMessage to `$LYX_PARENT` (`agent-naming`'s driver-parent-notice); the delivery prompt follows the same wording.
- The delivery prompt goes through the same live-session send the gate re-prompt uses (`gateRepromptText`'s path in `wait.go`).
- The review-output format is defined in `contracts/stencils/burler/burler-step-2-review.md`.
- `lyx loom approve`/`reject` (PR-Gate) are the idiom the new verbs mirror; they must not collide with them.

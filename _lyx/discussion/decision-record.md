# Parent review as a step in Discussion-Write's gate

## Goal

Port millhouse's `orch-review`/`orch-wait` into Discussion-Write's own producer gate.
When the writer reports done and the mechanical `discussion` validator passes, a reviewer (the run's parent, `$LYX_PARENT`) reviews the discussion pair once.
A reject re-prompts the still-live writer with the review, so the writer fixes it with its own context.
The next arrival re-runs the mechanical validator and then lets the run through to the Discussion-Review segment as today.
No new recipe row and no separate fixer.

Alongside it, Discussion-Write and the reviewer both read the board, so a design fits planned work and neither duplicates nor contradicts it.

## Scope

In:

- A third gate-result state, **pending**, in `shuttleengine`'s gate contract, and the wait loop's handling of it.
- A new gate name `parent-review` in `shedrecipe`'s gate vocabulary, and the Discussion-Write row's `gates` list gaining it after `discussion`.
- The review exchange on disk: a per-round request directory holding the request, the delivery record, the verdict and the review file, committed to weft with the discussion.
- Delivery of a one-line notice to the reviewer's reed strand by full name, hub-wide.
- A `lyx loom review` verb subtree: `notify`, `approve`, `reject`.
- A reviewer brief stencil the request renders, pointing at the review-output format and the board.
- Status: `lyx loom status` and `lyx batten status` show a run waiting on its reviewer and who it is.
- A loom config key for the review wait bound.
- Discussion-Write's stencil: read the board for design context, and handle a parent-review re-prompt.
- The orch start stencil: review requests are handled through a one-shot fork.
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
In the wait loop, a pending entry sends no re-prompt and changes no count.
The loop keeps polling and re-evaluates the gate list on each poll tick without waiting for a new arrival, while liveness checks and the run deadline keep running.
`finalize` never waits: an entry still pending at the final verdict is reported `waiting` and counts toward `Passed` like a let-through entry.
Bound: only a `PassOnCap` entry may return pending.
A pending result from any other entry is a gate error, so pending can never let a must-pass gate through.
`shedrecipe` rejects `parent-review` without `pass_on_cap: true` as a construction error naming the row.

Rationale: a closure that blocks for minutes would stall the wait loop's liveness and deadline checks.
A pending state keeps the existing loop in charge of time and lets a driver restart re-attach to a gate that is still waiting.

### D3 — The review exchange is files in a per-round request directory

Decision: each review round has one directory, holding:

- the request: run slug, round number, reviewer full name, opened-at time, the discussion pair's paths, and the rendered reviewer brief;
- the delivery record: delivered-at time, or the last failure reason;
- the verdict: approve or reject, recorded-at time, and whether the gate has consumed it;
- the review file, copied in by the verdict verb.

The directory sits beside the Discussion-Review run directory under the run root, never under `_lyx/discussion/`, which `discussionparser` owns.
It is committed to weft with the discussion pair at the discussion commit seam.
A fresh Discussion-Write run (a start, not an attach) opens the next round number; an attach continues the latest round.

Rationale: files survive a driver restart, so `AttachGated` finds the open request and keeps waiting rather than asking twice, and the committed review is the durable record of what the parent said.

### D4 — The `parent-review` closure

Decision: on each call the closure reads the current round's directory and returns:

- no parent recorded in the run's seed: passed, logged once at Info;
- no request yet: opens the request, attempts delivery (D5), returns pending;
- request open, no verdict, wait bound not reached: retries an undelivered notice at most once a minute, returns pending;
- wait bound reached with no verdict: passed, with a Warn naming the reviewer and the request path, and the request marked expired;
- verdict approve: passed;
- verdict reject, not yet consumed: marks the verdict consumed, returns failed with findings naming the copied review file;
- verdict reject, already consumed: passed.

The wait bound is measured from the request's opened-at time, so a restart does not reset it.
Bound on the consumed marker: if the driver dies between the closure marking a reject consumed and the re-prompt being sent, the writer never sees that review, and the next attach passes; the independent Discussion-Review rounds still run.

Rationale: the closure is idempotent over the on-disk state, which is what the gate-list commit asks of a request that must not repeat across an attach.

### D5 — Delivery types one notice line into the reviewer's reed strand

Decision: the gate resolves the reviewer's full name hub-wide, across every worktree's reed state, the way `lyx reed list` enumerates them.
It then types one line into that strand: the run slug and the request path, plus an instruction to review with a one-shot fork.
It types only when the provider's idle probe on that strand passes, the same probe `orch`'s watcher uses, so a running turn or an operator's draft is never typed over.
An unknown name, an ambiguous name or a parent that is not a reed strand fails delivery: the failure is logged at Warn and recorded in the delivery record, and the run keeps waiting until the wait bound.
Delivery never refuses or stops the run.
The provider-specific idle probe stays behind the existing provider seam; `reedengine` and `shuttleengine` gain no Claude specifics.

Rationale: the gate is Go and cannot call Claude Code's `SendMessage`; reed delivery by full name is what the Agent Name Invariant set up names for.

### D6 — `lyx loom review` verbs

Decision: a new `review` subtree under `lyx loom`:

- `lyx loom review notify [<slug>]` re-sends the open request's notice, bypassing the once-a-minute throttle but not the idle probe.
  It refuses when no request is open, naming that only Discussion-Write's gate opens one.
- `lyx loom review approve [<slug>] [--review <file>]` records an approve verdict; the optional review file is copied in as a record only.
- `lyx loom review reject [<slug>] <review-file>` records a reject verdict and copies the review file in.
  It refuses a missing or empty review file.

`<slug>` is optional inside a task worktree and required from the hub's prime worktree, where the verb resolves the task worktree through the hub layout.
`approve` and `reject` refuse when no request is open, and when the open request already has a verdict or has expired.
Every refusal names its way forward and gets a row in `contracts/specs/refusal-spec.md`'s loom section with a reaching test.
The verbs do not check who runs them; any session or the operator can submit.

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
The Discussion-Write stencil gains a short paragraph describing this re-prompt.
Discussion-Review's independent rounds read that entry with the rest of the support log.

Rationale: the writer keeps its context and may disagree, and the support log is where the review history already lives and what the next reviewers already read.

### D9 — Status shows the wait

Decision: while a round has an open request without a verdict, `lyx loom status` reports the run as waiting on its reviewer, naming the reviewer's full name, the open time and whether the notice was delivered, in place of a plain `running`.
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

Rationale: that is the context-cost property `orch-review` had.

## Constraints

- **Shuttle Provider-Seam Invariant:** the idle probe used for delivery stays behind the provider seam; neither `shuttleengine` nor `reedengine` references Claude specifics.
- **Agent Name Invariant:** names are resolved through reed strand state only, using `internal/agentname` for matching; nothing parses names elsewhere.
- **Told-Geometry Invariant:** the closure, delivery and status hook receive every path as told; no engine imports `lyxcwd`.
- **Shed Verb-Set Invariant:** the waiting-on-reviewer status is a told hook in `shedverbs`'s `Spec`, never a reimplemented `status` body.
- **Transient Stop Invariant:** pending is not a verdict and never carries the transient mark.
- **Refusal Way-Forward Invariant:** every new loom refusal names its way forward and lands with its refusal-spec row and reaching test in the same commit.
- **Fabric Git Invariant:** the request directory is committed by Go at the discussion commit seam through `fabricengine.ScopedPathspec`, never by an agent.
- **Discussionparser Sole-Parser Invariant:** the request directory is not under `_lyx/discussion/`.
- **Gate Self-Check Parity Invariant:** `parent-review` is not a mechanical gate and gets no self-check verb; the invariant's list stays as is.
- **Stencil Ownership Invariant:** the reviewer brief is a stencil read at call time.
- **Live-Substrate Spawn Observability:** delivery starts no process; any tmux call it makes goes through reed's existing logged paths.
- **CLI/Cobra Invariant:** every new command has a non-empty `Short`, JSON errors, and a `clihelp.ShouldAbort` check; help-tree tests cover the new subtree.
- **Documentation:** `contracts/specs/shed-recipe-spec.md`'s gates paragraph (new name, pending, the `pass_on_cap` requirement), `contracts/specs/loom-status-spec.md`, `docs/overview.md`'s loom row, and the touched package docs land in the same commit.

## Auto-mode assumptions

- The parent is normally the hub, hosted as a reed strand by `lyx orch start`; a hand-started hub gets no notice and sees the wait only through status.
- 60 minutes is a sensible default wait bound.
- One round per Discussion-Write run is enough; the independent segment reviews carry any further iteration.
- A notice typed into an idle Claude pane is read as the next user turn.
- An approve carries no actionable finding; a review with findings is submitted as a reject.

## Open risks

- **Typing race with the orch watcher.** The watcher and delivery can both see the hub idle on the same tick and both type, landing a notice inside a handoff turn.
  The request stays open and visible in status, and `lyx loom review notify` re-sends.
- **Hand-started hub.** A parent that is not a reed strand never receives a notice, so every run waits out the full bound unless the hub watches `lyx batten status`.
- **Lost reject on a crash.** The consumed-marker window in D4 can drop one review.
- **Idle-probe drift.** The probe reads provider pane state; a Claude UI change can make delivery wait until the bound.

## Acceptance criteria

- A Discussion-Write run whose seed has a parent opens a request after the `discussion` gate passes, delivers one notice to the parent strand, and shows as waiting on that reviewer in `lyx loom status` and in the prime's `lyx batten status`.
- `lyx loom review approve <slug>` from the prime lets the run through to Discussion-Bouncer with no re-prompt.
- `lyx loom review reject <slug> <file>` re-prompts the live writer once with findings naming the copied review file.
  The next arrival re-runs `discussion`; a passing arrival is let through without a second review, and a failing one re-prompts on the `discussion` budget as today.
- A run with no parent in its seed, or with `attempts: 0`, passes the review entry without opening a request.
- A request with no verdict past `parent_review_wait_min` passes with a Warn, the request is marked expired, and later verdict verbs refuse with a way forward.
- A driver restart while waiting re-attaches to the same round: no second request, no second notice once delivered, and the wait bound continues from the original open time.
- An attach after a consumed reject passes the review entry.
- A fresh Discussion-Write start after a completed round opens the next round number.
- Delivery to an unknown, ambiguous or non-reed parent logs a Warn, records the reason, and leaves the run waiting; `notify` re-sends.
- Delivery never types while the target's idle probe fails.
- A pending result from a non-`PassOnCap` entry is a gate error; `shedrecipe` rejects `parent-review` without `pass_on_cap: true`.
- The wait loop runs liveness and the run deadline while an entry is pending; a deadline expiry reports the entry `waiting` and finalizes.
- Each new refusal of the `review` verbs has a refusal-spec row and a test reaching it: no open request, verdict already recorded, request expired, missing or empty review file, slug required from the prime, unknown slug.
- The request directory is in the discussion commit.
- The Discussion-Write stencil tells the writer to read `lyx board list` and how to address a parent-review re-prompt; the reviewer brief points at the review-output format rather than restating it.
- Unit tests cover the closure's state table with a fake clock and a fake delivery seam, the wait loop's pending handling, recipe resolution, and the hub-wide name resolution.
  An integration test drives a gated run through request, reject, fix and let-through with a fake reviewer submitting through the verbs.

## Notes for the plan writer

- The gate list landed in `b7c478620`; `internal/shuttleengine/gate.go` and the gated branch of `wait.go`'s loop are where pending goes.
- `internal/shedrecipe/entries_gate.go`'s `resolveGateClosure` is the gate vocabulary; the run's parent lives in the seed (`shedrun.Seed.Parent`) and reaches `Env` from there.
- Hub-wide strand enumeration exists in `internal/reedcli/list.go`; per-worktree name lookup and typing are `reedengine`'s `ResolveStrandGUID` and `SendText`.
- `orch`'s watcher uses an idle probe through its `Session` seam; delivery should reuse that probe, not add a second one.
- The review-output format is defined in `contracts/stencils/burler/burler-step-2-review.md`.
- `lyx loom approve`/`reject` (PR-Gate) are the idiom the new verbs mirror; they must not collide with them.

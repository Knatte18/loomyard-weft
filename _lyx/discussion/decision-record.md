# Decision record — parent-review-wait

## Goal

Discussion-Write's parent-review wait costs less and keeps its promises.
A reject that arrives after a long review resumes the writer on a warm prompt cache.
The wait bound ends the wait even when the writer never ends its turn, including after a driver restart.
The gaps the PR #336 review left in the gate are closed, and a rewrite after a parent reject is never let through unreviewed.

## Scope

In: D1 to D11 below.
The writer is the Discussion-Write session, or the seats chair when Discussion runs on seats; advisors are out, since they take no turn after the chair's record exists.

Out:

- Mid-turn gate sends after a driver restart: built by `shuttle-send`, landed as commit e12fb6afb ("Shuttle sends land or retry"), which confirms a boundary idle in fact before every gate send and clears it on a later turn start.
- Turns the operator types into the writer: left to that same turn-start and cleared-boundary handling.
- The reviewer brief's scope-only judging stays unchanged: the Discussion-Review perch reviews the detail right after the parent, and a broader parent review would raise orch spend.
- What wakes the orch and who resumes a halted run (`orch-wake-policy`), who reviews (`alignment-agent`), and per-task overrides of the reject cap (`config-layers-run-settings`).
- Moving the gate's sends off the writer-mediated SendMessage hop: planned in `creel` stage 3; D1 and D4 work over the current hop.

## Decisions

### D1. The gate pings the idle writer inside its prompt-cache lifetime while a delivered request waits, at most 3 times per round

Decision:
While the latest round's request is open with no verdict and its delivery is recorded, the parent-review gate closure returns pending with a keep-alive line as its `Send` once the keep-alive interval has passed since the latest of the round's delivered-at, last-prompt and last-ping times.
The interval is told to the gate as a `GateConfig` duration, whose zero turns pings off; the cap of 3 is a constant in `internal/parentreview`.
`shuttleengine.Engine` gains a provider-neutral method returning how long a session's prompt cache stays warm while idle for a given `Spec` and `Config`; `claudeengine` answers it from the role's resolved prompt-cache TTL, so no cache lifetime is spelled outside it.
The loom wiring sets the interval to the writer's lifetime less one minute, 4 minutes at the 5m default, and to zero when the lifetime reaches the wait bound, so a writer whose role is mapped to `1h` is not pinged at the default 60-minute bound.
The round's `delivery.json` gains a ping count and a last-ping time, recorded under the round lock when the closure carries a ping, so the clock and the cap survive a driver restart.
A carried ping counts toward the cap whether or not its send lands.
The line is a new one-line stencil, `loom-template-parent-review-keepalive.md`, rendered by `internal/loomengine` with the ping's number and told to the gate as a render seam beside `RenderDelivery`; it tells the writer to change nothing, run nothing and end its turn.
The ping number makes each ping's text distinct, so shuttle's pending-send record never mistakes a ping for an earlier one already sent.
The discussion stencil's "Messages the gate may send after you end your turn" section, which the seats chair also receives, names the keep-alive as a third message kind that changes nothing.
The loop sends it through the existing pending-send path, so it lands only at an idle boundary through `sendWithin`.
A ping's pending result carries its own failed-send way-forward: nothing needs doing, and the next ping retries while the cap allows; it never names `lyx loom review notify`, which would re-send a notice the reviewer already has.

Bound:
A ping is carried only while a delivered request is open with no verdict, so it stops at a verdict, at expiry and at finalize.
At most one per interval and 3 per round; after the third the cache is left to expire.
No ping is carried while delivery is unrecorded, or while the reviewer-liveness hold is in force, since a reviewer with no live session yields no verdict the ping could keep the cache warm for.
A ping spends no gate prompt and no waiting notify.

Rationale:
At the 5m TTL a ping re-reads the writer's context from cache at about a tenth of the price, while a cold resume re-writes it at about 1.25 times, so three pings cost about a quarter of one cold resume.
At a one-in-three reject rate that pays off for reviews up to about 16 minutes, which covers a normal fork review.
Without the cap, a review that runs to the 60-minute wait bound would spend 14 pings, more than the cold resume they save.
Mapping the discussion role to `1h` through `claude_prompt_cache_ttl_roles` would need no ping, but bills every cache write of the writer's whole session at about 2 times instead of 1.25, about three quarters of the context extra on every run whether or not a reject comes; three pings cost at most 0.3 of it, and only on runs whose review outlasts the interval.

### D2. Wait expiry is checked off the boundary, from the store

Decision:
`shuttleengine.GateEntry` gains an optional expiry probe, a read-only closure reporting whether the entry's wait has run out.
While the writer is not at an idle boundary, the wait loop calls the probe of every entry that has one on the throttled cadence of D3, whether or not the loop has seen the entry pend, so a loop that attached to a writer stuck mid-turn after a driver restart still reaches it.
When a probe reports expired, the loop finalizes `OutcomeDone`, and finalize's evaluation reaches the entry's `Final` closure when every earlier entry passes; `Final` marks the open request expired and passes with the existing timeout Warn, or reads a verdict that landed meanwhile.
The parent-review wiring sets the probe to "the latest round's request is open, has no verdict and is older than the wait bound"; it reads the round's files without the round lock, since `Final` re-reads them under it.
The `wait.go` doc that counts finalize call sites names this one; it finalizes a positive outcome, so it needs no `allOutputFilesExist` check.

Bound:
The probe sends nothing, counts nothing and writes nothing; it reports that the parent-review wait has run out whatever the tree's state.
Finalize evaluates the entries once in list order, with no re-prompt, and a reject that raced the probe goes through `Final`'s existing terminal path.
When the mid-turn tree fails the discussion's mechanical entry, finalize stops at that entry and never reaches `parent-review`'s `Final`: the run ends `Stuck` with the request left open and unexpired and no timeout Warn, and Discussion-Write, which has no `on_stuck`, halts it where a boundary would have re-prompted the writer.
That finalize removes the writer's strand like every `OutcomeDone`, so the next `lyx loom start` is a fresh spawn: the request stays open and unexpired in its round, and the spawn works on the next round, which `PrepareRound` begins, since the gate reads only the latest round.
A verdict the parent records for the halted round after that start refuses until the gate opens the next round's request, and lands on that request after.
That is accepted: it is the end the run deadline's finalize already reaches for a writer stuck mid-turn, reached sooner, and a writer that never ends its turn cannot be re-prompted either way.

Rationale:
Expiry needs no send, so waiting for a turn end held a stuck writer, for example one on a permission prompt for `lyx loom review delivered`, to `discussion_timeout_min` instead of `parent_review_wait_min`.
The probe keeps `shuttleengine` free of parent-review knowledge and marks nothing: only the parent-review closures mark a request expired, at a boundary or in `Final`.

### D3. Gate re-evaluation without a new arrival runs at most every 5 seconds

Decision:
The wait loop re-runs a pending gate at an idle boundary with no new arrival, and the D2 probes off the boundary, at most once per 5 seconds, a constant in `internal/shuttleengine`.
A new turn-end arrival and a restored cleared boundary still evaluate at once.

Bound:
The throttle only delays a re-read by up to 5 seconds; it never skips an arrival and never extends the run deadline.

Rationale:
At the 500 ms poll every pending tick re-ran `discussionparser.Validate`, three locked JSON reads and a `MkdirAll`; a verdict read 5 seconds late costs nothing.

### D4. Neither the orch nor its reviewer fork replies to the writer

Decision:
The parent-review brief and the orch role stencil's Parent-review section each gain one line: the verdict reaches the writer through the gate only, so no message is sent to the writer, neither an acknowledgement nor a reply to its notice.

Rationale:
A reply starts a writer turn the wait loop did not start, which costs a turn and moves the writer off the boundary while a verdict may be pending.
The brief is what the fork reads, and the orch role is what the orch reads before forking, so each carries the rule for its own reader.

### D5. The review verbs name the prime's status command for a slug-addressed run

Decision:
The no-open-request refusal and the supersede refusals of `lyx loom review` choose by whether a slug was given, not by where the verb runs: with a slug they name `lyx batten status <slug>` run from the prime, and with none, which only a task worktree addressing itself accepts, `lyx loom status`.

Rationale:
`lyx loom status` takes no slug, so it cannot address another worktree's run from the prime, where the orch runs these verbs, or from a sibling task worktree.

### D6. `approve --review` refuses a missing or empty file

Decision:
The store refuses an approve with a review path whose file is missing, unreadable or whitespace-only, with `ErrEmptyReviewFile`, exactly as it refuses a reject; an approve with no `--review` is unchanged.
The refusal spec's row for this refusal names the verb that was run, not `reject`.

Rationale:
The refusal spec already promises this, and an empty review copied into the round is a record of nothing.

### D7. `parent-review` must be the last entry of its gate list

Decision:
`shedrecipe`'s gate resolver refuses a gate list in which a `parent-review` element is followed by another element, naming the row and the way forward: move `parent-review` to the end of the list.

Rationale:
A must-pass entry after a pending `parent-review` would read `not_reached` at a deadline finalize while the review is pending.

### D8. `delivered` after a verdict writes nothing

Decision:
`lyx loom review delivered`, with or without `--failed`, on a round whose verdict is recorded succeeds without writing `delivery.json` or logging the failure Warn, and its result says the round is settled.

Bound:
It skips only the delivery stamp of a round that can no longer use it; an open round with no verdict is stamped as before, and an expired round still refuses.

Rationale:
After an approve the discussion commit may already have run, so a late stamp left `_lyx` dirty until the next status commit.
A refusal would send the writer looking for a fix to a step that has nothing left to do.

### D9. A fresh spawn reuses a latest round that holds no request

Decision:
The store's fresh-spawn preparation returns the latest round unchanged when its directory holds no request, besides the existing reject and superseding-approve cases.

Rationale:
A run with no reviewer, or a spawn that died before its gate opened a request, left an empty `round-N/` behind, and every later fresh spawn added another.

### D10. Every rewrite after a reject is reviewed; the cap halts

Decision:
The parent-review gate drops its pass-at-cap mode, reversing commit 518dc7400's "the rewrite goes on to the perch": a reject below the cap goes to the writer and its rewrite opens the next round, and a reject at the cap halts the run `blocked`, as it already does without the mode.
`shedrecipe` refuses `pass_on_cap` on a `parent-review` element, naming the way forward: drop the key, and raise `attempts` for more review rounds.
The loom recipe's `parent-review` element sets `attempts: 2` with no `pass_on_cap`, so the first reject's rewrite is reviewed once more, and a second reject halts the run with the gate's existing terminal message (`lyx loom review approve` from the parent, then start the run again).
The recipe comment, the `internal/parentreview` package doc and the `reject` verb's help are updated to match.
A run in flight across the binary change whose one reject is recorded but whose rewrite has not yet passed the gate reads that reject as below the new cap on resume, so its discussion goes back to the parent as round 2; a run already past Discussion-Write keeps its unreviewed rewrite, since it never evaluates the gate again.

Bound:
This removes a let-through, never adds one; the generic `pass_on_cap` of other gate entries is unchanged.

Rationale:
In `daemon-wakeups` (2026-10-10) the only reject's rewrite went on to Plan with no parent review.
With one attempt and no pass-at-cap the first reject would halt before the writer saw it, so two attempts is the smallest cap under which every rewrite is reviewed.
The board entry asks for this trade against the orch spend a second round costs.

### D11. This run's own parent review is the live proof of the hop

Decision:
No smoke test is added.
The PR #336 live check is met by this run as far as its records reach: its round-1 `delivery.json` must carry a delivered-at, and the round's `verdict.json` a verdict recorded after that stamp.
The landing description cites both files and says what they leave open: neither records who stamped the delivery, that a SendMessage to `ly:orch` preceded the stamp, who ran the verdict verb or which notice prompted it.

Rationale:
Every loom run in this hub with the orch as parent takes the hop, and this run's records show a delivery stamp and a verdict after it.
They do not isolate the hop: the `delivered` verb checks no caller and accepts a stamp with no send, and batten's own waiting-review note is a second route to the orch; that gap is an Open risk.
An llm-tier test of a two-session exchange is expensive to keep for a path production exercises on every run.

### Added after Discussion (operator, 2026-10-11): Parent review stays scope-only

Decision: Parent review checks the discussion against the board entry's scope and against other board entries only; it does not judge design quality.

Rationale: Discussion-Review and the twill lens forks already judge quality; only the parent sees the board, and a second quality judge could reject what the Bouncer approved.

### Added after Discussion (parent, 2026-10-11): No expiry on resume after a D2 halt

Decision: D2's Bound sentence 'A resume of the halted run marks that request expired ... at its first boundary' is void: the halt finalizes OutcomeDone and removes the writer's strand, so any later start is a fresh spawn; round N's request stays open and unexpired and the new spawn works on round N+1. The plan builds no expiry-on-resume path and pins no test for one. Expiry is marked only by the parent-review closures, at a boundary (closures.gate) or in Final; the probe marks nothing.

Rationale: Round-3 Discussion-Review F1 and F2: finalize in internal/shuttleengine/wait.go cleans up on OutcomeDone with KeepPane false, so no attach reaches gate()'s expiry branch; closures.gate in internal/parentreview/gate.go marks expiry at a boundary, so finalize is not the only place.

## Constraints

- `PATTERN-pane-text-injection`: the ping is typed only through the gate's pending `Send`.
- `PATTERN-stencil-ownership`: the keep-alive line is a stencil read at call time through `internal/stencilstore`.
- `PATTERN-producer-pointer-rule`: the discussion stencil names the keep-alive message without restating its text.
- `PATTERN-told-geometry`: the probe and the ping clock live in `internal/parentreview` over its told store, inside the import set its `doc.go` declares; `internal/shuttleengine` learns only an optional closure.
- `PATTERN-shuttle-provider-seam`: the prompt-cache lifetime behind D1's interval is answered only by `claudeengine`, through the provider-neutral `Engine` method; `internal/parentreview` and the loom wiring hold no cache lifetime.
- `PATTERN-refusal-way-forward`: D5, D6, D7 and D10's refusals each name the way forward, land their spec row and get a test; D7 and D10's are recipe-build refusals, reached by a developer editing the recipe.
- `PATTERN-daemon-wakeups`: the loop's poll interval is unchanged; D3 only skips work inside a tick.
- `PATTERN-completion-signal`: D2 finalizes `OutcomeDone` through finalize's own evaluation, never a negative outcome.
- `PATTERN-documentation-lifecycle`: `internal/parentreview/doc.go` and the `shuttleengine` gate docs describe the built behaviour in the same commit.
- `PATTERN-test-speed` and `PATTERN-test-economy`: clocks are injected; new cases extend the existing gate, store, shedrecipe and loomcli tests where a table fits.

## Auto-mode assumptions

- D1's cap rests on an unmeasured reject rate near one in three and reviews mostly under 16 minutes; its one-minute margin under the cache lifetime is the writer's pick.
- D10's `attempts: 2` is the writer's pick.

## Open risks

- A ping the writer answers with more than a short reply, or with a tool call, costs more than estimated; the stencil asks for neither.
- D2 removes the writer's strand mid-turn on expiry; a writer mid-way through a hand-requested discussion edit is cut off, with the halt D2's Bound states when its tree fails the mechanical entry.
- D11's records do not show who stamped the delivery or which notice the orch acted on, so the hop is not isolated from batten's waiting-review note; reject this record if an isolating check, such as the orch's transcript, is wanted before landing.
- At the cap the writer never sees the second review's findings; the parent's approve lets the rewrite on as it stands, so the second review's findings must be ones the Discussion-Review perch can carry.
- Two review rounds per run where one used to suffice raise orch spend on runs whose first review rejects.

## Acceptance criteria

- With a delivered open request and no verdict, no ping is carried before the interval since the latest delivered-at, last-prompt or last-ping time, one is carried at the interval, the next one interval after that, and none after the third in the round.
- The interval is 4 minutes for a writer at the 5m TTL, and a writer whose role is mapped to `1h` gets no ping at the default 60-minute bound.
- After a driver restart the ping clock and count resume from `delivery.json`, so a round never gets more than 3 pings in total.
- No ping is carried before delivery is recorded, while the reviewer-liveness hold is in force, after a verdict, or after expiry; a ping leaves the round's prompt count and waiting notifies unchanged; a ping whose send fails still counts.
- Two consecutive pings carry different text, and a failed ping's Warn names no `lyx loom review notify`.
- A writer that stays mid-turn while a request is pending past the wait bound ends the run within one throttle period: `done` with the request marked expired and the timeout Warn logged when the tree passes the mechanical entry, and `Stuck` with the request left open and no timeout Warn when it fails it.
- The same holds when the loop attached to a writer already mid-turn after a driver restart, with no boundary seen.
- A verdict recorded between the probe and finalize is read: an approve passes and a reject takes the terminal path.
- The probe never fires before the wait bound and never for a request with a verdict.
- With no new arrival, a pending entry's closure and the probes run at most once per 5 seconds, and a turn-end arrival is evaluated at once.
- The reviewer brief and the orch role stencil each say that no message goes to the writer.
- With a slug, from the prime or from a sibling task worktree, the no-open-request and supersede refusals name `lyx batten status <slug>` run from the prime; with no slug, in the task worktree, they name `lyx loom status`.
- `approve --review` with an empty, whitespace-only or missing file refuses and records no verdict.
- A gate list with an entry after `parent-review` refuses to build with the way forward; `parent-review` last builds.
- `delivered` and `delivered --failed` on a round with a verdict exit 0, write no `delivery.json`, and say the round is settled; on an expired round they still refuse.
- Two fresh spawns with no reviewer leave one round directory, and a fresh spawn after a reject still reuses the reject's round.
- A `parent-review` element with `pass_on_cap` refuses to build with the way forward.
- With `attempts: 2`, a first reject re-prompts the writer and the rewrite opens round 2; a second reject halts the run `blocked`, and `lyx loom review approve` lifts it.
- This run's round-1 `delivery.json` carries a delivered-at, its `verdict.json` a verdict recorded after it, and the landing description cites both with the gaps D11 names.

## Notes for the plan writer

- The pending re-evaluation and the boundary handling sit in `internal/shuttleengine/wait.go`'s `Wait` loop and `handleGatedBoundary`; `GateEntry` is in `internal/shuttleengine/gate.go`.
- The gate closures are in `internal/parentreview/gate.go`, the round store, `Delivery`, `PrepareRound` and `RecordVerdict` in `store.go`.
- The gate list resolver is `internal/shedrecipe/entries_gate.go`; the recipe is `contracts/recipes/loom-recipe.yaml`; `parentreview.GateConfig` is built in `internal/loomcli/wiring.go`.
- `shuttleengine.Engine` is in `internal/shuttleengine/engine.go`; `claudeengine` resolves a role's TTL in `resolvePromptCacheTTL` (`command.go`).
- The review verbs and their way-forward strings are in `internal/loomcli/review.go`; the delivery and brief renderers in `internal/loomengine/parentreview.go`; refusal rows in `contracts/specs/refusal-spec.md`.
- Stencils: `contracts/stencils/loom/loom-template-parent-review-brief.md`, `loom-template-discussion.md` (included by `loom-template-discussion-chair.md`), and `contracts/stencils/orch/orch-template-role.md`; `contracts/stencils/claims_test.go` and `markers_test.go` may pin their claims and markers.
- New top-level tests may need a row change in `cmd/lyx/testdata/test-budget.yaml`.

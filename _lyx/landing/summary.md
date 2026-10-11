# Parent-review wait: keep-alive pings, no pass-at-cap, bounded wait

Discussion-Write's parent-review wait now costs less and keeps its promises.
A reject that arrives after a long review resumes the writer on a warm prompt cache.
The wait bound ends the wait even when the writer never ends its turn, including after a driver restart.
The gaps the PR #336 review left in the gate are closed, and a rewrite after a parent reject is never let through unreviewed.

## What changes

- **Keep-alive pings.**
  While a delivered request is open with no verdict, the parent-review gate sends the idle writer a one-line ping once the keep-alive interval has passed since the latest of the round's delivered-at, last-prompt and last-ping times, at most 3 per round.
  The ping tells the writer to change nothing, run nothing and end its turn; its text is the new stencil `loom-template-parent-review-keepalive.md`, and the discussion stencil names it as a third gate message.
  The ping count and last-ping time are recorded in the round's `delivery.json` under the round lock, so the clock and the cap survive a driver restart.
  A carried ping counts toward the cap whether or not its send lands, and a failed ping's way-forward never names `lyx loom review notify`.
  No ping is carried before delivery is recorded, while the reviewer-liveness hold is in force, after a verdict, or after expiry.
- **Interval from the cache lifetime.**
  `shuttleengine.Engine` gains `PromptCacheLifetime`, which `claudeengine` answers from the role's resolved prompt-cache TTL.
  The loom wiring sets the interval to that lifetime less one minute (`keepAliveMargin`), and to zero, which turns pings off, when the lifetime reaches the wait bound.
- **Wait bound off the boundary.**
  `shuttleengine.GateEntry` gains an optional read-only expiry probe.
  While the writer is mid-turn the wait loop calls every entry's probe, so a writer that never ends its turn, or one the loop attached to after a driver restart, is finalized `done` once the request is older than the wait bound.
  Finalize's `Final` closure then marks the request expired with the existing timeout Warn, or reads a verdict that landed meanwhile.
- **Re-evaluation throttle.**
  A pending gate with no new arrival, and the probes, run at most every 5 seconds (`gateRecheckDue`); a turn-end arrival or a restored cleared boundary still evaluates at once.
- **No reply to the writer.**
  The parent-review brief and the orch role stencil each say that the verdict reaches the writer through the gate only, so no acknowledgement or reply is sent.
- **Every rewrite after a reject is reviewed.**
  The gate drops its pass-at-cap mode; `shedrecipe` refuses `pass_on_cap` on a `parent-review` element, and `loom-recipe.yaml` sets `attempts: 2`.
  A first reject re-prompts the writer and its rewrite opens round 2; a second reject halts the run `blocked`, and `lyx loom review approve` lifts it.
- **Review verb fixes.**
  - The no-open-request and supersede refusals name `lyx batten status <slug>` when a slug was given and `lyx loom status` when none was.
  - `approve --review` refuses a missing, unreadable or whitespace-only file with `ErrEmptyReviewFile`, as `reject` does.
  - `delivered` on a round with a verdict exits 0, writes nothing and says the round is settled; an expired round still refuses.
- **Round store and recipe.**
  A fresh spawn reuses a latest round that holds no request instead of adding an empty `round-N/`.
  The gate resolver refuses a gate list with an element after `parent-review`, naming the move to the end.

`internal/parentreview/doc.go`, the `shuttleengine` gate docs, the refusal spec and the shed-recipe spec describe the built behaviour.
`cmd/lyx/testdata/test-budget.yaml` raises the `shuttleengine` row and others the new tests push past.

## Design decisions

- The cap of 3 pings is a constant in `internal/parentreview`.
  At the 5m TTL three pings cost about a quarter of one cold resume, and they pay off for reviews up to about 16 minutes at an assumed one-in-three reject rate; uncapped, a review running to the 60-minute bound would spend more on pings than the resume saves.
  Mapping the discussion role to `1h` instead would bill every cache write of the whole session at about 2 times rather than 1.25.
- The expiry probe lives in `internal/parentreview` and `shuttleengine` learns only an optional closure.
  The probe sends, counts and writes nothing; only the parent-review closures mark a request expired.
- When the mid-turn tree fails the discussion's mechanical entry, finalize stops at that entry and never reaches `parent-review`'s `Final`.
  The run ends `Stuck` with the request left open and unexpired and no timeout Warn, and Discussion-Write halts where a boundary would have re-prompted.
  Finalize removes the writer's strand, so the next `lyx loom start` is a fresh spawn working on the next round; a verdict recorded for the halted round after that start refuses until the gate opens the next round's request.
  This is the end the run deadline's finalize already reaches for a writer stuck mid-turn, reached sooner.
- `attempts: 2` is the smallest cap under which every rewrite is reviewed: one attempt with no pass-at-cap would halt on the first reject before the writer saw it.
  The generic `pass_on_cap` of other gate entries is unchanged.
- Parent review stays scope-only; it checks the discussion against the board entry's scope and other board entries, and does not judge design quality.

Closes nothing on GitHub; the board entry lists no issue.

## To check by hand

- The first review's `delivery.json` and `verdict.json` for this task were not available to this description, so the live check of the writer-mediated SendMessage hop from PR #336 is not confirmed here.
  Check that the round-1 `delivery.json` carries a delivered-at and the round's `verdict.json` a verdict recorded after it.
  Neither file records who stamped the delivery, that a SendMessage to `ly:orch` preceded the stamp, who ran the verdict verb or which notice prompted it; the `delivered` verb checks no caller, and batten's waiting-review note is a second route to the orch, so the hop is not isolated.
- D2's Bound overstates the probe's reach: a finding left open from review (`d2-bound-overstates-probe-reach`).
  Confirm the doc and the code agree on when the probe finalizes `done` and when the run ends `Stuck`.
- The test for the 5-second throttle duplicates cases the gate tests already cover (`card12-throttle-test-duplicates`); check `internal/shuttleengine/gate_pending_test.go` for a case that can fold into an existing table.
- The refusal tests for `lyx batten status <slug>` add a second table beside an existing one (`card5-second-refusal-table`); check `internal/loomcli/review_test.go`.
- The slug-versus-no-slug choice of the status command in the review refusals has no test of its own (`d5-slug-choice-untested`); check `internal/loomcli/review.go`.
- A run in flight across this binary change whose one reject is recorded but whose rewrite has not yet passed the gate reads that reject as below the new cap on resume, so its discussion goes back to the parent as round 2.
- The cap rests on an unmeasured reject rate near one in three, and a ping answered with more than a short reply costs more than estimated.
- At the cap the writer never sees the second review's findings; the parent's approve lets the rewrite on as it stands.
  Two review rounds per run where one sufficed raise orch spend on runs whose first review rejects.
- A mid-turn expiry removes the writer's strand, so a writer in the middle of a hand-requested discussion edit is cut off.
- Verify failures fixed along the way: `battencli.TestWire_EverySeamFilled` and three `envkit` tests failed because `ParentReview.RenderKeepAlive` was nil after wiring and missing from the nil-legal seam list in `internal/testkit/envkit/envkit.go`; commit `30a88e0cc` adds it, and `lyx gate test --tags integration ./internal/battencli ./internal/testkit/envkit` passed.

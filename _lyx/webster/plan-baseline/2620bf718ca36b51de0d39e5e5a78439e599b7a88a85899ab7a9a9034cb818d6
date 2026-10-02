# Card 6 — review-gate-hold

**Edit:**
- `internal/parentreview/gate.go`
- `internal/parentreview/gate_test.go`
- `internal/parentreview/doc.go`

**Intent:**
Hold the delivery prompt while the reviewer has no live session, so the gate's prompt budget is not spent during an orch relaunch (Decision 4).

`GateConfig` gains `ReviewerLive func() (bool, error)`, the told liveness seam; nil means the reviewer is treated as live, which is how a legacy-name reviewer with no known worktree is wired.

On an open request with no verdict, after the wait-bound check (so an expired request still passes with its Warn), the gate asks `ReviewerLive` before carrying any prompt, gate-driven or notify-driven.
When the reviewer is not live it returns pending with no `Send`, records no prompt, and leaves `Prompts` and `WaitingNotifys` untouched, so neither `maxGatePrompts` nor a waiting notify is spent.
Opening a new request writes the request and brief first, so the wait bound starts, then applies the same check before its first prompt.
Once the reviewer is live, the gate prompts as today.
It logs one Info line per round that it is holding, naming the slug, reviewer and round; the closures remember the last round they logged for.
A `ReviewerLive` error counts as not live: the gate holds and logs a Warn naming the error, and never returns it as a gate error.
A pending entry is evaluated on every poll tick, so that Warn is bounded like the Info line: once per round per distinct error text, remembered beside the last-logged round.

Bound: the hold delays the delivery prompt and never skips the review short of `WaitBound`; a reviewer that never becomes live ends in the existing timed pass.

Update `GateConfig`'s doc, the gate closure's doc and `doc.go`'s description of `delivery.json`'s prompt count.

Tests in `gate_test.go`: with a not-live seam, opening a request returns pending with an empty `Send` and `Prompts` 0, and repeated arrivals keep it so; flipping the seam to live makes the next arrival return the delivery prompt and count it; a nil seam prompts at once; a seam returning an error holds without a gate error and Warns once across repeated arrivals with the same error; a held request still expires and passes at the wait bound; a waiting notify is not consumed while held and is carried once live.

**ImpactSummary:** Adds an optional liveness seam to the parent-review gate; with the seam nil the gate behaves as after card 5.

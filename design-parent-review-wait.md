# Parent-review wait: warm writer cache on reject, wait bound off the boundary, gate hardening

`parent-review-wait` · Task · enhancement, loom, shuttle, orch

# Part 1: keep the waiting writer's cache warm

## Seen (2026-10-11)

While Discussion-Write's parent-review gate waits, the writer or seats chair and its advisors stay alive and idle (`internal/shuttleengine/wait.go:284-299`, `372-383`; advisors kept by `KeepPane: !isChair`, `internal/seatengine/prompt.go:161`, stopped only in `join`, `internal/seatengine/engine.go:245-252`).

- On approve nobody takes another turn: the writer's strand is removed without a message (`wait.go:385-389`) and advisors are killed (`engine.go:251`), so the cache does not matter.
- On reject the findings are typed into the same idle session (`wait.go:391-399`, `internal/parentreview/gate.go:165-170`).
  If the review took more than the 5-minute prompt-cache lifetime, that turn re-reads the whole writer context uncached, which is the expensive case.
- While delivery is unrecorded, the gate re-sends the delivery prompt up to three times at least a minute apart (`gate.go:20-22`, `190`); each re-send is a writer turn that may land on a cold cache.

## Idea

Keep the waiting writer's cache warm, the way the Burler fixer already does: `lyx burler await` caps each wait below the 5-minute lifetime so every re-run keeps the fixer's cache warm (`internal/burlercli/await.go:25`).
The operator suggests a ping injected into the idle writer about every 4 minutes while the gate waits.

- A ping is a small cached turn; it pays off only when a reject is likely, so weigh its cost against the cold re-read on reject.
- Alternative: on a reject that arrives after the cache has expired, start a fresh session with only the decision record, the support log and the findings instead of resuming the old context.
- Advisors need no ping: they never take another turn after the chair's record exists.

# Part 2: gate hardening

## Goal

The parent-review gate holds its promises in the cases the first version left open: the wait bound fires even when the writer never ends its turn, no gate send lands inside a writer turn after a restart, and a live hand-started hub is proven to receive the notice.

## From the PR #336 review (2026-10-02)

- **Live smoke.** The smoke test uses a stub writer that records the typed line, so the SendMessage hop to a real hand-started parent has never run; run it once live.
- **Wait bound off the boundary.** Expiry is checked only at a writer turn boundary (`internal/shuttleengine/wait.go`, `gateAtBoundary`), so a writer stuck mid-turn (for example on a permission prompt for `lyx loom review delivered` in interactive mode) holds the run until `discussion_timeout_min` instead of `parent_review_wait_min`; expiry needs no send, so evaluate it on every tick.
- **Mid-turn sends after a restart** moved to task `shuttle-send` (2026-10-08).
- **Turns the loop cannot see.** A turn started by the parent replying to the writer through SendMessage, or by the operator typing, leaves the loop at a boundary; tell the orch fork not to reply to the writer.
- **Poll cost.** While pending at a boundary every 500 ms tick re-runs `discussionparser.Validate`, three locked JSON reads and a `MkdirAll`; throttle re-evaluation to a few seconds.
- **Way-forward from the prime.** The no-open-request refusal names `lyx loom status <run>`, which cannot address the child from the prime; name `lyx batten status` there.
- **`approve --review` and an empty file.** The refusal spec says it refuses a missing or empty file, the code only a missing one; align them.
- **Entry order.** `shedrecipe` does not require `parent-review` to be the gate list's last entry; a must-pass entry after it would be `not_reached` at a deadline finalize while the review is pending.
- **`delivered` after a verdict** can write `delivery.json` after the discussion commit and leave `_lyx` dirty until the next status commit.
- **Round numbers** grow on every fresh spawn with no reviewer (empty `round-N/`, never committed).
- **A rewrite after a parent reject goes unreviewed.** In `daemon-wakeups` (2026-10-10) the orch rejected the Discussion once, the parent-review cap was then reached, and the rewrite went on to Plan with no parent review; a rewrite after a reject always gets one more parent review, or reaching the cap escalates instead of passing silently.
- **Review scope.** The brief says "Do not re-review the design's internal quality", narrower than millhouse's `orch-review`; confirm with the operator.

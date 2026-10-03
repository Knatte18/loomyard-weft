# Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke

`parent-review-hardening` · Notes · feature

## Goal

The parent-review gate holds its promises in the cases the first version left open: the wait bound fires even when the writer never ends its turn, no gate send lands inside a writer turn after a restart, and a live hand-started hub is proven to receive the notice.

## From the PR #336 review (2026-10-02)

- **Live smoke.** The smoke test uses a stub writer that records the typed line, so the SendMessage hop to a real hand-started parent has never run; run it once live.
- **Wait bound off the boundary.** Expiry is checked only at a writer turn boundary (`internal/shuttleengine/wait.go`, `gateAtBoundary`), so a writer stuck mid-turn (for example on a permission prompt for `lyx loom review delivered` in interactive mode) holds the run until `discussion_timeout_min` instead of `parent_review_wait_min`; expiry needs no send, so evaluate it on every tick.
- **Mid-turn sends after a restart.** `AttachGated` replays events from offset 0, so the first tick treats the writer as idle even mid-turn: a notify-carried prompt or a reject re-prompt can be typed into a running turn, and a consumed reject is let through while the writer is still fixing.
- **Turns the loop cannot see.** A turn started by the parent replying to the writer through SendMessage, or by the operator typing, leaves the loop at a boundary; tell the orch fork not to reply to the writer.
- **Poll cost.** While pending at a boundary every 500 ms tick re-runs `discussionparser.Validate`, three locked JSON reads and a `MkdirAll`; throttle re-evaluation to a few seconds.
- **Way-forward from the prime.** The no-open-request refusal names `lyx loom status <run>`, which cannot address the child from the prime; name `lyx batten status` there.
- **`approve --review` and an empty file.** The refusal spec says it refuses a missing or empty file, the code only a missing one; align them.
- **Entry order.** `shedrecipe` does not require `parent-review` to be the gate list's last entry; a must-pass entry after it would be `not_reached` at a deadline finalize while the review is pending.
- **`delivered` after a verdict** can write `delivery.json` after the discussion commit and leave `_lyx` dirty until the next status commit.
- **Round numbers** grow on every fresh spawn with no reviewer (empty `round-N/`, never committed).
- **Review scope.** The brief says "Do not re-review the design's internal quality", narrower than millhouse's `orch-review`; confirm with the operator.

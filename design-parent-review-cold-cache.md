# Parent-review wait lets the idle writer cache expire before a reject

`parent-review-cold-cache` · Note · enhancement, loom, shuttle

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

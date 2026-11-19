Lens fix-proposal: I challenged the proposals behind F15, F16 and F17 as the fixer carried them out, and found no defects.

No findings.

- **F15 (settle and parent-findings check in `settleUnlessRefused`):** I ran the cases that could break it, and each holds.
  - Re-judge after a refusal: `judgeCall` finds the held, done ask through `RoundAsk` and sends no second one. The judge's await then answers done at once.
  - Settling before `routeDecisions` matches what `judgeCall` did before the move.
  - A segment whose seat has `AskBeforeJudge` false, and a Bouncer with no seat, return 0, so the check and the settle do nothing.
  - Round and seed are the key, and each segment has its own run directory, so one segment's ask cannot match another's.
  - The `standsAsRecorded` replay still settles directly. That is correct, because the parent's decision settles over that round.
- **F16 (fragment instead of embedded const):** each worry I built is covered.
  - Pins: `stencilstore.Pins` runs over `stencils.Registry()`, so the new fragment is pinned automatically (`internal/loomcli/stencilpins.go:23`).
  - Seeding: seeding follows the registry entry the fixer added.
  - Read failure: it degrades through `renderFailureReason`. The ask has already gone out by then, but the next call finds it held and sends no duplicate.
  - Engine: `stencilstore.Read` takes no engine, so the review's "for the judge's engine" has nothing to bind to.
  - Prompt text: the `TrimSpace` plus `"\n\n" … "\n"` rebuild gives the same prompt shape as before.
  - Docs: `internal/shedadapters/doc.go:626-630` still describes the behaviour correctly and never named the Go constant.
- **F17 (`judgeStartedAt` set in `runAgent`):** `tableWithParent` has a single production caller, the table closure inside `runAgent` (`verify.go:887`), so the start is always set before the table is built. Resume and attach still start a fresh deadline, as the field comment states. The new test fails with the defect in place, because a zero start gives no cap and a 120-minute deadline.

Tool calls (2):
1. Grep for stencil pin callers and Bouncer `Parent:` wiring in `internal`, plus the parent lines of `internal/shedadapters/doc.go`. My context lacked the pin-list enumeration and any doc mention of the moved check.
2. Read `internal/shedadapters/doc.go` lines 618-640, plus a grep for stale refusal wording. I needed to know whether the doc described the old `judgeCall`-only refusal.

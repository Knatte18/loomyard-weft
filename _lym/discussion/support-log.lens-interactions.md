```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state in the creel-hook-post decision record. Two pairs break. One acceptance criterion is under-specified as a result.

**Findings**

1. **BLOCKING — D4 × D5: a ring is followed by a typed bell.** No live waiter exists between a ring and the woken turn's own Stop.
   - When the waiter rings, it exits 2 and its connection closes, so the daemon now sees no live waiter.
   - The mail is still claimable, because D4 claims nothing.
   - The strand stays idle until Claude starts the woken turn, which took about 0.3–0.4 s in spike case 3. The worker's next probe at the 1 s base can fall in that gap, or in any later idle moment before a new Stop waiter registers.
   - D5 then types the bell into the pane on top of the wake.
   - D5's "once per new claimable set" counts only typed bells, not rings, so nothing stops it.
   - Fix: a ring counts as the bell for that claimable set, and D5 waits for that set until the next Stop registers a waiter or a boundary claims the mail.
   - The acceptance criterion "An idle strand with a live waiter is woken … message text then shows … as the Stop block" does not exclude this double wake.
2. **MEDIUM — D2 × the 8-continuation cap (Constraints): mail is lost and a turn end goes unrecorded.**
   - On a ninth consecutive block with no tool call between, Claude Code overrides the block and ends the turn.
   - D2's merged command has already claimed the mail, which is acknowledged after the write and journaled as read. It has also already skipped the `Stop` record.
   - So the overridden turn ends with no turn-end record in `events.jsonl`, which `ParseEvents` needs. The claimed mail may never reach the model, because the override discards the block, but creel marks it read.
   - D2's bound says the left-over mail "is reached by D4 or D5". The mail in the overridden block is not left over: it was claimed.
   - Fix: D2 reads `stop_hook_active` and the cap, or claims nothing and records the turn end once a block would hit the cap.
3. **LOW — D4 × D2/D1: the rewake turn may count toward the cap.**
   - A rewake turn starts with `stopHookActive: true` in the bundle code the writer read, so it may count toward the cap.
   - Wake turn plus mail turns without a tool call bring the cap closer. The record does not mention it.

**Pairs checked**

| Pair | Result |
|---|---|
| D1 × D2, the shared claim and reservation | held: whichever claims first delivers, the other finds nothing |
| D1 × the 8 KiB budget × `reason` + `systemMessage` duplicated | held: the budget applies to the lines, and the JSON carries them twice |
| D2 × `ParseEvents`, a blocked stop with no record | held, except at the cap (finding 2) |
| D2 × D4, both `Stop` entries firing at a blocked stop | held: the new waiter registers, the idle check sees a running turn, and the next real Stop supersedes it |
| D2 × D3 | held: settings `Stop` does not fire in a subagent |
| D2 × hold and pause, non-owner mail with the flag standing | held: `hook-read` prints nothing and the turn end is recorded |
| D2 × `EventWaiting`, a stop with forks outstanding | held: the block continues the main thread |
| D3 × D4, the waiter's subagent skip | held |
| D4 × D5, the live-waiter state after a ring | broken (finding 1) |
| D4 × D5, waiter timeout and TERM, then the bell | held |
| D4 × D5, a stall other than a draft | held: neither rings or types, and typing would fail anyway |
| D4 × the cap | weakly broken (finding 3) |
| D5 × D6, the worker types only injects, commands and the bell | held |
| D5 × a draft (guarded send `ErrBusy` versus D4's draft tolerance) | held: D5 waits and D4 rings |
| D6 × D1, the notice reservation removed | held |
| D7 × D4/D5, no Stop after an interrupt | held: D5 covers it |
| D7 × pause, interrupt claims into the pause, then the post | held: nothing is delivered until resume |
| D7 × the existing rule "no delivery while the interrupt runs" | held |
| D8 × the parent-directive LOOM rule and stencils | held: the rule is worded by program, not line shape; D11 covers `orchcli/doc.go` |
| D8 × D9, `from "EXTERNAL"` versus "mail from EXTERNAL" | held |
| D9 × D6, operator-level trust for agent mail | held: the bound is stated and sourced to the operator |
| D9 × D5, the typed bell begins with `[creel]` | held: benign text |
| D9 × D10 | held: same rule |
| AC "a ring that loses to a running turn delivers nothing" × D4 | held: trivially true |
| AC "under a hold only the owner's mail is claimed or belled" × D4/D5/D2 | held |
| Constraint `PATTERN-tolerant-records` × D6 | held |

Tool calls made: none.

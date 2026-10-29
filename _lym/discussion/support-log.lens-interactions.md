```yaml
lens: interactions
outcome: reported
```

**Lens: interactions.** I checked every pair of decisions that act on the same state, and each acceptance criterion and constraint against the decisions it touches.

## Findings

**MEDIUM — D3 and the Constraint "Each message is still delivered once": the record does not say what happens when the stderr write succeeds but the acknowledgement fails.**
- D3 covers two failure paths: a write that fails, and a connection that drops before the acknowledgement. Both release the mail unread.
- D3 does not say what `hook-wait` does when the stderr write returned but the acknowledgement then fails, for example because the daemon died or the socket closed.
- If `hook-wait` still exits 2, the model sees the mail, the daemon releases it unread, and the next tool-boundary claim, Stop claim or bell hands it over again. That is a double delivery, which breaks the once-only constraint.
- If it exits 0 instead, Claude Code shows nothing and the bell path delivers later, so the mail arrives once.
- D1 has the same split, because `hook-read` "exits 0 exactly when it wrote a block, whatever follows the write" (`internal/creelcli/doc.go`). That case is pre-existing, but D3 is a new path and should pick its exit rule explicitly.
- Fix: D3 should state that `hook-wait` exits with the wake code only after the acknowledgement is confirmed, and exits 0 otherwise. It should also bound the reverse case: an acknowledgement journaled whose wake then never reaches Claude Code.

**LOW — D3 and its acceptance bullet: "the ring stays unanswered" conflicts with the existing rule that marks a ring answered when a claim hands out mail.**
- Today `ClaimAtBoundary` marks the strand's last ring answered as soon as a claim hands out at least one message, before any acknowledgement (`internal/creelengine/verbs.go` around line 349).
- If the ring's claim reuses that path, a release after hand-out leaves the ring already marked answered. The 30-second unanswered-ring window then never applies. The bell comes from the 5-second no-waiter grace instead.
- The mail is still not stranded, but D3's failure claim and its acceptance bullet ("lets the bell's unanswered-ring window fire") would be false.
- Fix: D3 should say the ring's claim marks the ring answered only on acknowledgement. Alternatively, state that the 5-second grace path is the one that applies.

## Pairs checked

- D1 × D2 (same `hook-read`, branch on event; merged Stop command's exit-0 rule): held.
- D1 × D3 (Stop count): the ring resets the count to 0. The woken notification is enqueued with `stopHookActive` true, so its Stop does not reset the count again, and the cap of 6 is still below Claude Code's 8. Held.
- D1 × D3 (no-turn reading after a ring, which needs a turn end after the ring): a delivering Stop still records no turn end, and a Stop with no mail exits 1 and records one. Held.
- D1 × D5 (Stop cap and unknown-count reasons logged at Debug): held.
- D2 × D5 (Debug for empty claims; claims run only while the flag stands): held.
- D2 × the 8 KiB constraint: two copies of the lines, which matches the pre-`c49ba448` PostToolUse output size, so no regression. Held.
- D3 × D4 (stderr lines behind the `rewakeMessage` prefix, which opens with `[creel]` and matches the trust sentence): held.
- D3 × D5 (`acknowledged` / `released` outcomes on the ring path): held.
- D3 × the bell (same set is never rung again; bell after the window; a bell-started turn ends at a Stop that D1 delivers): held, subject to the LOW finding.
- D3 × the existing waiter-reconnect rule (ring response lost, waiter re-registers, same set, bell after 30 s): held.
- D3 × the "no ring on empty claim" bullet: the claimable set and the claim come from the same lock and rules, and the first message is always handed out. Held.
- D3 × Constraint "Each message is still delivered once": broken; see the MEDIUM finding.
- D3 × D1 race (ring vs an operator-started turn): the ring fires only when no turn is in progress, and reservations are exclusive. Held.
- D4 × `PATTERN-shuttle-provider-seam` (the keys live in claudeengine settings): held.
- D3 × Out of scope "no journal record change": the ring's acknowledgement writes the existing `read` record. Held.
- Acceptance D1, D2, D4 and D5 against their decisions: consistent. The D3 bullets are covered by the two findings.

## Tool calls

1. Grepped `internal/creelengine/deliver.go` and `verbs.go` for the answered and wake-mark bookkeeping. My context did not hold the code that marks a ring answered.
2. Read `wakeHookStrand` in `internal/creelengine/deliver.go` and the idleness source in `internal/creelcli/delivery.go`. I needed to know whether a failed ring leaves the strand readable as idle so the bell can fire.

# Card 7 — ring-echo

**Edit:**
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Reading`
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#Reading.Release`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine#Store.claimLocked`
- `internal/creelengine#Store.flaggedFilesLocked`
- `internal/creelengine#Response`
- `internal/creelengine#daemon.handOut`
- `internal/creelengine_test#TestScheduler_HookWake`
- `internal/creelengine_test#TestStore_MailFlags`
- `internal/creelengine/doc.go`

**Create:**
- `internal/creelengine#EchoHeading`

**Uses:**
- `internal/creelengine#Store.FailJournalWrites`
- `internal/creelengine_test#newFlaggedEnv`
- `internal/creelengine#Store.Forward`
- `internal/creelengine_test#env.start`

**Intent:**
Decision D6, daemon side: the ring's mail reaches the operator as an echo at the woken turn's first claim.

State: the struct `Store` gains `echoes map[string][]echoItem`, an unexported in-memory map from a strand to its owed echo items, oldest first, each item `{id int64; line string; reserved bool}` (the new unexported type `echoItem` in `verbs.go`), made empty by `OpenStore`; a daemon restart drops it (D6's Bound).
`EchoHeading` is the line `hook-read` puts above the echoed lines; its doc says the echo reaches the operator alone.

Ring acknowledgement: `Reading.Ack` of a ring reading appends to the strand's echo one item per id it passed, in order, whether that id was journaled read or skipped as no longer the holder's unread mail, so a full Ack echoes every line and an Ack whose journal write fails echoes the ids before the failing one and none from it on (those card 3-ack-failure-releases released).

Claim: the struct `Reading` gains `Echo []string`, the echoed lines handed out, and the unexported `echoIDs []int64`.
`Store.ClaimAtBoundary` reads the pause first as today, but a pause no longer returns before the locked step: in it, the claim first hands out the strand's unreserved echo items oldest first, reserving them, then the mail through `Store.claimLocked` unless a pause, the Stop cap or an unknown Stop count keeps the mail back; a hold narrows the mail and never the echo.
Under a pause the mail steps are skipped whole, the Stop count's reset, cap check and count included, so a paused claim leaves the count as today's early return does.
`claimLocked` measures the budget over the echo lines already in `reading` as well, so the echo and the mail share the 8 KiB budget, the first item, echo or message, always handed out, and what does not fit stays owed.
Only handed-out mail marks the ring answered and counts a Stop; an echo alone continues no turn.
A ring hands out no echo, so its claim measures the budget over mail alone, and with an echo owed it can hand out mail the boundary claim's shared budget would leave for later; card 4-ring-claims-mail's "the set a boundary claim would hand the same strand" holds from here on for the mail with the echo set aside.
`Reading.Ack` of a claim drops its echo items, whether or not its journal writes succeed, since the hook already showed them; `Reading.Release` unreserves them, so they stay owed.

Flags: `Store.flaggedFilesLocked` also asks for the flag of a strand with an owed echo item, reserved or not, so the tool-boundary hook and the merged Stop command run hook-read while an echo is owed; Ack and Release call `syncFlagsLocked` themselves whenever they change a strand's echo, since an echo-only claim journals nothing and `Store.do` syncs only after an append.

Protocol: the struct `Response` gains `Echo []string` tagged `json:"echo,omitempty"`; `daemon.handOut` sets it from `reading.Echo` and asks an acknowledgement when the reading holds ids or echo items.

Tests (tier0), in `TestScheduler_HookWake` (with card 4-ring-claims-mail's `registerTaking`):
- `AnAcknowledgedRingLeavesAnEchoTheNextClaimHandsOutAheadOfItsMail`: ring of #1 acked, a post #2; a tool-boundary claim's `Echo` is #1's line and `Messages` #2; after its Ack a claim hands out nothing.
- `ThePausedOrHeldStrandsClaimStillHandsOutTheEcho`: after the acked ring, with a pause set the claim's `Echo` is #1's line and `Messages` empty, and that claim is released, so the echo stays owed; with the pause cleared, another sender's hold and a non-owner post, the next claim's `Echo` is #1's line and `Messages` empty.
- `TheStopCapKeepsTheMailBackButNotTheEcho`: a short ring acked, then bigBody posts; six Stop claims, the first with `StopHookActive: false` and the rest `true` as `StopBlocksAreCappedAtSixPerTurnAndANewTurnStartsTheCountOver` passes them, each released, hand out the echo and one message and raise the count; the seventh Stop claim hands out the echo alone.
- `TheEchoSharesTheBudgetAndAReleaseKeepsIt`: a bigBody ring acked and a bigBody post; the claim hands out the echo alone; after its Release the next claim hands out the echo again.
- `AJournalFailureAtTheRingsAckEchoesOnlyTheIdsBeforeTheFailingWrite`: a ring of three short messages, `FailJournalWrites(1)`, Ack fails; after `FailJournalWrites(-1)` a claim's `Echo` is #1's line and `Messages` #2 and #3.
- `AJournalFailureAtAClaimsAckDropsTheEchoItHandedOut`: echo owed and a post #2; the claim hands out both; `FailJournalWrites(0)`; Ack fails; after removing the fault a claim hands out #2 and no echo.
- `ARingAckEchoesAnIdThatLeftTheHolderBeforeTheAck`: a ring of #1 and #2, short; before its Ack alpha forwards #2 to beta with `Store.Forward`; the Ack journals #1 and skips #2, and the next claim's `Echo` holds both lines.
In `TestStore_MailFlags`, the row `AnOwedEchoKeepsTheFlagUntilTheClaimThatShowsItIsAcknowledged`: on a flagged env whose strands the row re-scripts with `e.del.SetStrands(creelfake.HookStrand(alpha), creelfake.LiveStrand(beta))` before `e.start`, since `newFlaggedEnv` scripts pane-mail strands that are never rung, alpha is rung through a marked `HookWait` with one short message and acked; the flag stands with no unread mail; the echo claim's Ack clears it. Then two posts of 5000 bytes each, two of whose lines pass the budget, and a marked ring of the first, acked: the flag stands for the second, which the ring's budget left unread, and for the echo; the echo claim (the echo alone, by the budget) acked leaves it standing for the second; the claim of the second acked clears it.

`internal/creelengine/doc.go` gains D6: the echo, its order ahead of mail, pause, hold and Stop cap, the shared budget, its drop and keep rules, the ack-failure ordering, the flag, and both Bounds (the echo shows after the model has the mail, a StopFailure claims nothing and a session that ends first never shows it; a daemon restart drops it, and D5's `ring` line records the ids).

**ImpactSummary:** In-memory echo state on the Store and two protocol fields, all within creelengine; the claim's rules change only for a strand owing an echo, and an older client ignores the unknown key.

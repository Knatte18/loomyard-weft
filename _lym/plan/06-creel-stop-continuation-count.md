# Card 6 — The claim knows its boundary and counts a turn's Stop blocks

**Create:**
- `internal/creelengine#Boundary`
- `internal/creelengine#stopBlockCap`

**Edit:**
- `internal/creelengine#Store.ClaimAtBoundary` -> `func (s *Store) ClaimAtBoundary(sender Sender, boundary Boundary) (*Reading, error)`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Request`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine_test#TestScheduler_Reminders`

**Intent:**
D2: the Stop hook blocks while claimable mail stands, so creel counts, per strand, the consecutive Stop blocks of the strand's current turn, and stays below Claude Code's cap of 8 so no override ever discards a block whose mail creel journaled read.

`Boundary` names which hook asks: `Stop bool`, true for the turn-end Stop hook and false for the tool-boundary hook, and `StopHookActive bool`, the Stop hook input's `stop_hook_active`.
`Request` gains `Stop bool` (JSON `stop`, omitempty) and `StopHookActive bool` (JSON `stop_hook_active`, omitempty), read by the claim verb alone; `daemon.dispatch` passes them to `ClaimAtBoundary` as a `Boundary`.
An old hook-read that sends neither claims as a tool boundary, as it did.

`Store` gains `stopBlocks map[string]int`, the Stop blocks of each strand's current turn; a strand with no entry counts as at the cap.
`OpenStore` starts it empty, so a daemon that starts, a restart after a binary replacement included, holds every strand at the cap and no Stop blocks until one of the resets below; the count is never journaled.
`ClaimAtBoundary`, with its checks before the claim unchanged, applies a reset before it checks the cap:
a Stop whose `StopHookActive` is false sets the strand's count to 0, since Claude Code starts that turn's count at 0;
a Stop at or above `stopBlockCap` claims nothing and returns an empty reading, so the turn end is recorded;
a Stop that hands out at least one message adds one to the count, before the client writes it, so the count errs early, never late;
a tool-boundary claim that hands out at least one message sets the count to 0.

Every caller of the re-signed `ClaimAtBoundary` is in this card: `daemon.dispatch`, the `claimLines` helper of `TestStore_ClaimAtBoundary`, which claims as a tool boundary, and the three claims inside `TestScheduler_Reminders`, which keep claiming as a tool boundary.
`TestStore_ClaimAtBoundary` gains rows, each posting bodies of 5,000 bytes so the 8 KiB budget admits exactly one message per claim, and enough of them that mail still stands when the cap is reached, so a Stop that claims nothing does so because of the cap and not an empty mailbox:
a fresh store's Stop with `StopHookActive` true claims nothing though mail stands (the daemon-start cap), and the same Stop after a tool-boundary claim hands out mail;
a Stop with `StopHookActive` false hands out mail, five more Stops with it true each hand out one message, and the seventh claims nothing while mail still stands;
after that capped Stop, a Stop with `StopHookActive` false hands out mail again.
The doc's tool-boundary section gains the count and its resets.

**ImpactSummary:** ClaimAtBoundary is re-signed; its one production caller is the daemon's claim dispatch and its test callers sit in two creelengine tests, all in this card.

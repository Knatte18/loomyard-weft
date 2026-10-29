```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each claim and bound in the record and support log.

**Findings**

1. MEDIUM — D3, and the Constraint "Each message is still delivered once": the record does not say what `hook-wait` does when the write succeeds but the acknowledgement fails.
   - Counterexample: the daemon is replaced (its binary-swap exit) or the connection drops after `hook-wait` wrote stderr but before the ack's confirming line. The daemon releases the reservation, so the mail stays unread with its flag standing.
   - If `hook-wait` still exits 2, Claude Code wakes the model with the mail. The woken turn's first tool call or Stop then claims the same mail again, so the model reads it twice.
   - Claude Code reads an asyncRewake hook's stderr only after the process exits, and wakes only on exit 2 (`if(Ae.code===2)…Bin(...)`). So `hook-wait` can avoid the duplicate by exiting 0 when the ack fails.
   - D3 should state that rule. The same gap exists in today's `hook-read`, whose doc says it "exits 0 exactly when it wrote a block, whatever follows the write". The record's once-guarantee should either note that existing exception or close it.

2. MEDIUM — D1, its Rationale and the Open risks: whether a Stop's additional context alone keeps the turn going is proven only for Claude Code 2.1.312, and nothing guards another version.
   - Counterexample: a host runs a Claude Code version whose Stop path does not push additional context into the continuation list. The mail is claimed, acked and journaled read, the hook exits 0, the merged command records no turn end, and the turn ends with the mail delivered to no live turn.
   - The block design that D1 replaces continued the turn in every version. The new risk is version-dependent, not only the mode-check risk the record names.
   - Open risks should name this, with a bound such as: the tier3 smoke run is re-run on each Claude Code upgrade.

3. LOW — D3, "claims … in the same locked step as the ring, under every rule of a boundary claim: hold, pause".
   - Today `Store.ClaimAtBoundary` reads the pause through `s.delivery.Paused` before `s.do`, outside the store lock. A pause seam call cannot sit in the locked step, so the pause cannot be checked atomically with the ring.
   - Counterexample: a `lym tender` pause starts between the probe's pause read and `ringHook`'s locked claim. The ring then hands mail to, and wakes, a strand the pause covers.
   - The window is the same as today's ring wake, but now it also delivers. D3 should say the pause is read just before the locked step, as a boundary claim reads it, and state that window.

4. LOW — D3's Bound names only "a session that dies between the hook's exit and Claude Code reading its stderr".
   - It misses a second loss window: Claude Code ends the hook after the ack but before its exit. That can happen at the 86400 s hook timeout, at session end, or with `/clear`. If Claude Code then sees no exit 2, the mail is journaled read and never shown.
   - The window is tiny (one ack round trip and its journal fsync), but the record states a bound and this path is outside it.

5. LOW — D4's Bound, "a version without the keys ignores them".
   - This is verified only for 2.1.312, whose schema has the keys. It is not shown that an older or newer schema strips unknown hook keys rather than refusing the settings file.
   - If a version refuses the file, every hook in it is lost, recording and creel delivery included, not only the label.
   - Recording the claim as an assumption about other versions, or covering it with the same smoke check as finding 2, would bound it.

**Claims attacked**
- D1, the turn continues on additional context alone (2.1.312): held, apart from the mode check the record already names. Broken across versions (finding 2).
- D1, the Stop cap of 6 holds under Claude Code's cap of 8: held. Claude Code resets its count on every tool-use turn and at turn start (`pi` sets `stopHookBlockingCount:0`). The daemon resets only on `stop_hook_active` false or on a tool-boundary claim that hands out mail. So the daemon's count never falls below Claude Code's.
- D1, the next Stop's `stop_hook_active` is true: held, since the continuation guard sets `stopHookActive:!0`.
- D1, the operator sees the mail once and the model once, with no command text: held.
- D1, an exit 0 after output records no turn end: held by the merged command's `&& exit 0`.
- D2, PostToolUse additional context is not drawn in the terminal: held, since `hook_additional_context` is in `LT`, the list of attachments the terminal does not render.
- D2, the turn goes on whatever the hook prints: held, since no `continue:false` is printed.
- Constraint, 8 KiB fits every field: held. The claim is at most 8192 bytes, which is at most 8192 UTF-16 units, under the 10,000-unit field cap. Claude Code's per-field cap of 16384 and its 1 MiB total cap both hold even with D2 repeating the lines in two fields.
- D3, the rewake text is not truncated: held as far as `srt` and `Bin` show.
- D3, a ring whose claim is empty does not ring: held, since the claimable set is computed under the lock. The pause is the exception (finding 3).
- D3, a live session shows what the hook wrote: held, with the gaps in findings 1 and 4.
- D3 and the bell's 30-second window for an unanswered ring: held.
- D4, `rewakeMessage` and `rewakeSummary` are honoured from settings whatever the plugin: held (`H??…`, `We??G??…`). Unverified for other versions (finding 5).
- D5, its levels and outcomes: held.
- Constraint, each message is delivered once: broken (finding 1).

**Tool calls**
- Read the bundle around the 16384 field cap and the output-size function, because the context lacked what caps a hook field and the whole output.
- Grepped the bundle for `stopHookBlockingCount`, because the context lacked where Claude Code resets its Stop block count.
- Read the `LT` attachment list and its use, because the context did not show that the list means "not rendered in the terminal".
- Printed the 16384 cap's use further on, because the first read cut off before showing it.
- Read the tool-use continuation's guards, because the context lacked whether a tool turn resets the block count.

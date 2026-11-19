```yaml
lens: fix-proposal
outcome: reported
```

**Findings**

1. **MEDIUM**: `Store.ReplyWith` in `internal/creelengine/verbs.go` calls `s.slackRelayCheck` inside the `s.do` closure, which is the F1 proposal as written.
   - **Evidence:** `post` runs the same check before `storePost` and holds no store lock, so the bridge's file reads stay out of the store mutex. `slackBridge.RelayPostReady` reads the geometry, the beam listing through the prime's dobby start, the forwarding file and the relay lock.
   - **Effect:** every one of those reads now runs under the store mutex. A slow or hung read on a reply to relay mail stalls every other verb and claim.
   - **Fix:** resolve `original.From` under a short lock first (chase resolution included), release, run the check, then take the lock again to journal.
   - **Complication:** the original message can change between the two lock sections, so the holder check must run again.

2. **LOW**: `PostResult.Delivered` and `Response.Delivered` are set to `live`, so a stored post to a running relay answers `delivered: true`.
   - **Evidence:** the `protocol.go` comment says "so the mail is posted at once". The mail is only journaled, and the relay still skips mail older than `staleMailAge` or mail it fails to post.
   - **Case:** a live relay that cannot post answers `delivered: true` for mail that never reaches Slack. The board entry named `delivered: false` for the down case only.
   - **Fix:** either document the field as "the relay runs" or set it only when false.

3. **LOW**: `Reply` in `verbs.go` is a wrapper kept only for tests, and its comment points at `ReplyWith`.
   - **Evidence:** `ReplyWith` is the only production caller, in `server.go`.
   - **Cost:** the wrapper breaks the self-contained comment rule and the "delete the unused" rule in `code-quality`.
   - **Fix:** change the test callers, or accept the wrapper.

No problem found in the F2, F4, F5, F6, F7 and F8 proposals. I built these cases against them:
- **F2 rows:** the strand row (relay down, sender refused) still stores and answers `receiver_live` true, and the beam-program row answers nothing. Both prove the guard.
- **F4 wrapping:** `ForwardingOn` returns false with nil for a missing file, per the existing "button off" row, so only real read failures are wrapped.
- **F6 EXTERNAL row:** `IsOrchStrand("ly:EXTERNAL")` is false, so the row is reached.

**Tool calls:** none; my context already held every file I judged.

```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix made against HEAD.

No findings.

What I checked:
- **15 ticks:** `agedPersistInterval = time.Minute` (`internal/creelengine/deliver.go:22`), and `chaseInterval` is 15 minutes. Under the start-probe rule, every running tick charges one interval, so chase 1 at the 15th tick, chase 2 at the 30th and `unanswered` at the 45th are consistent. The same count holds for the 15th tick after a restart.
- **The F1 premise holds:** the `rec.Of == eventPosted` branch of `applyCarried` sets `p.programs[rec.From]` whenever `isProgramMailbox` is true (`fold.go:345-347`). `isProgramMailbox` reads the sender through `agentname.ParseProgram`, so "exactly as `applyPosted` does" describes the code correctly.
- **The new retention row can be reached:** `carryRecords` carries every message with `msg.origin < cutoffDay && msg.Status == StatusUnread` (`fold.go:418`). That includes an unread chase and an unread sender notice, and each keeps `To: msg.Holder`, so "the carried messages stay with their holders" holds. `checkReceiverLocked` (`pool.go:199-204`) refuses any name that is neither a listed strand nor in `programs`, so asserting that a post to `CREEL` is refused as an unknown receiver is consistent.

Calls made:
- Grepped for the `agedPersistInterval` value and the `carryRecords` carry condition; my context had the constant's name but not its value, and didn't have the carry condition.

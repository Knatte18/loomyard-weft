```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals carried out as written (F1, F4–F16) and found two problems.

Proposals I challenged:
- **Hold:** F4, F6, F7, F8, F9, F10, F11, F12, F13, F14, F15, F16.
- **Raised below:** F1 and F5.
- **Out of my lens:** F2 and F3, which were fixed differently.

**MEDIUM: the F1 failure count fires on every dobby tick, including the ticks right after a healthy spawn**
- **Changed text:** card 9, the restart-cap bullet: "A `whileAway` start that finds the relay down while a `LastSpawnAt` stands and no accepted first read was written after it … counts one failure … it then waits from one minute … before it spawns again". The matching `TestEnsureRelay` row says "five `whileAway` starts in a row with the clock moved past each wait".
- **The case:**
  1. The prime dobby ticks as often as once a second (`internal/dobbyengine/loop.go:25`, `wakeFloor = time.Second`).
  2. After a spawn, the relay sits in `AwaitReparent` (polling once a second, bound 30 seconds), then `ReapInherited`, then the lock wait. All that time `RelayLive` reads it as down, and `FirstReadAt` is still zero or earlier than `LastSpawnAt`.
  3. So the tick one second after a healthy spawn meets the counting condition. Nothing in the text limits counting to once per spawn, or to starts after `NextStartAt` has passed. The ticks during the one-minute wait meet the same condition again.
- **Result:** a healthy start can reach five failures and `gave-up` within seconds. Even if counting were once per spawn, every healthy start would count one false failure and push the next spawn back a minute.
- **Missing coverage:** the test row moves the clock past each wait, so it never ticks during a wait or during a relay's start.
- **What the rule needs:** count only at a start that is due (after `NextStartAt`), once per spawn, and only after a grace from `LastSpawnAt` that covers the reparent bound and the first read.

**LOW: the F5 lost-confirmation case gives no wait when no post succeeded**
- **Changed text:** card 8: "An `ErrWaitTimeout` after the handler ran is a lost confirmation …: the record stays, and `read` returns the floor wait when a post succeeded."
- **The case:** the handler ran and returned nil, but posted nothing. For example, every message was skipped from the record, was older than 24 hours, or came from a sender that is never posted. The confirmation then times out.
- **The gap:** the text gives no return value for this case. It is neither the empty read, whose doubled wait applies only when the handler did not run, nor a post that succeeded. Card 8 also has no row for it.

Calls I made beyond what my context held:
- I grepped and read `internal/githubclient/cache.go`'s `cacheDir`. My context lacked it, and I needed it to test whether F14's row (clearing `HOME`, `XDG_CONFIG_HOME` and `LOCALAPPDATA`) reaches the refusal on every OS. It does.
- I grepped `internal/dobbyengine`'s timing constants. My context lacked how often the prime dobby ticks, and the F1 per-tick case depends on it.

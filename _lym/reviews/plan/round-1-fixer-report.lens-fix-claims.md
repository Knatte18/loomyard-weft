```yaml
lens: fix-claims
outcome: reported
```

This lens checked the sentences the fix added or rewrote against the code at HEAD.

**[BLOCKING] Card 9 Intent, and card 10's paragraph on a landing the ledger lost.**
- **Changed text:** card 9 says "`PushPairAnchored` reads the records hold itself, so every push of a task pair meets it: … `lym fabric push` and its detached bypass through `fabricengine.DetachedPush` alike".
- **Summary:** the detached bypass never reaches `PushPairAnchored`, so a hold read placed there leaves that push unheld, and F1's premise stays unresolved for it.
- **Evidence:**
  - `internal/fabricengine/teardownlock.go:98-133`: `DetachedPush` returns `CoalescePushRequest(req, SyncOptions{}, here)`.
  - `internal/fabricengine/coalesce.go:116`: `CoalescePushRequest` calls `coalescePushSides(...)`.
  - `coalescePushSides` is called only from `coalesce.go:97` (`CoalescePushBothAt`) and `coalesce.go:116`, never from `PushPairAnchored`.
- **What it affects:** the hold read must also sit in `coalescePushSides`, or in a step both paths share. The same applies to `CoalescePushBothAt`'s path and to card 10's trailer read.

**[LOW] Card 12's `landLocally` paragraph.**
- **Changed text:** "a kill between the two leaves the commit local and unheld only until the resume's `LandLocally` records it, with no push having sent it."
- **Summary:** once the process dies the lock is released, so the claim holds only through card 10's lost-ledger trailer read, and card 12 never names it.
- **Evidence:** that read is the only thing that stops a push of a records commit with no ledger hold behind it. And it covers only the paths that go through `PushPairAnchored`, per the finding above.

**[LOW] Cards 11 and 17, "through `gitkit`".**
- **Changed text:** "An unreachable origin is the fixture's origin URL pointed at a path that does not exist, through `gitkit`", and card 17's "an unreachable prime origin, through `gitkit`".
- **Summary:** `gitkit` has no exported helper that repoints an origin URL.
- **Evidence:** `rewriteOriginURLInConfig` (`internal/gitkit/gitkit.go:261`) is unexported. The step can still be done with `gitkit.Git(t, dir, "remote", "set-url", ...)`, so the wording overstates what the kit provides but the step works.

I checked these and they hold:
- Heddle's `startSync` and the tender pass only the Bolt directory to `SpawnSync` (`heddlecli/cli.go:173`, `tendercli/deps.go:154`).
- `PushBeam` sits inside `hubLandDeps` (`beamcli/deps.go:92-519`).
- `TestCancelIntegration_KeptPairJudgeOnly` is tier1.
- `prkit` and `takeoverkit` exist and do what the cards use them for.
- `runpusher.Start` spawns nothing while the pusher lock is held (`start.go:58-59`).

Tool calls:
1. `DetachedPush`, the push-lock directory, the exported `gitkit` functions, the `prkit` and `takeoverkit` docs, the cancel test's tier and `hubLandDeps`' range: I needed the push route and the kit and test facts the cards cite, which my context did not hold.
2. `DetachedPush`'s body: it showed the bypass goes to `CoalescePushRequest`.
3. `CoalescePushRequest`'s body: it showed it delegates to `coalescePushSides`.
4. The callers of `coalescePushSides`: they confirmed `PushPairAnchored` does not go through it.

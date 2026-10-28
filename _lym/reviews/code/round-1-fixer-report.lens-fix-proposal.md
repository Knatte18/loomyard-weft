```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 and build the concrete case where each is wrong.

**Findings**

1. **LOW** — `internal/creelengine/chase.go` `stepChases` and `internal/creelengine/sweep.go` `releaseGoneHolder`: the F1 proposal reads the hub's strands unconditionally.
   - Summary: the listing is now taken on every call, even when no step is due or nothing is settled, so the listing cost moves out of the lock but is paid far more often.
   - Evidence:
     - `chaser.run` calls `store.stepChases` on every running tick (every `agedPersistInterval`), and the chaser's `probe` already lists strands on the same tick.
     - `stepChases` settles a message only when `msg.ChaseConsumed >= base+chaseInterval` and `msg.Chases >= maxChases`, which is rare, and only then does a strand sender's notice need the listing.
     - The new `listing := s.listStrands()` runs before `s.do` regardless, so each tick costs one extra tmux/reed listing per chaser.
     - `releaseGoneHolder` lists even when `chaseableLocked(holder)` is empty.
     - A lazier shape reads the listing only when a settle is about to happen. For example, `stepChases` could do a first pass under the lock to find whether any message is due to settle, then read and re-enter.
   - This is a cost, not a wrong result. Reading before the lock also means the listing can be slightly stale by the time it is used, which the notice's "still listed" check tolerates.

2. **LOW** — `internal/orchcli/notifierwiring_test.go`: the F2 proposal is carried out as a source scan, which is weaker than the behavior test the review asked for.
   - Summary: the test pins the text of the `SetNotifier` closure, not where the notice posts.
   - Evidence:
     - The review's `fix` asks for a test that "builds the runner's notifier for a held agent in a beam and asserts it posts to the strand `orchroute.Asker` resolves". The test only checks that `orchcli.NotifyAsker(` appears in the text up to the first `"\n\t})"`.
     - A closure that is reformatted so it no longer ends at `\n\t})`, or one that calls `NotifyAsker` and discards the result, would pass without the notice being routed correctly.
     - Reverting to `NotifyParent` does fail the test, so the stated regression is covered.
   - Both existing source-scan tests in the repo (such as `TestProductionFiles_NeverReferenceHubWatchdogMechanism`) set the precedent for this form, so I rate it no higher than LOW.

F3: no finding. The rewritten sentence is correct, and the removed clause names no symbol that other docs still depend on.

**Tool calls:** none. My context already held every file needed.

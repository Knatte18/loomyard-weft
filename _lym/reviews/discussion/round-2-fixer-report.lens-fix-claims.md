```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM: the "holder already has its slot" claim is false for the main capture.**
- **Changed text:** the D7 Rationale sentence "since the lock's holder already has its slot, a check waiting for the lock never blocks the holder, so concurrent checks … serialise without deadlock", and the D7 Decision sentence "in a gate slot and under the capture lock, taken in that order".
- **Problem:** the slot-before-lock order is stated only for current captures, "here and in D8". The main capture still takes the lock first and its slot second. It is also not removed: D14 drops only `capture --side`, so `lym drift capture` stays.
- **Evidence:**
  - `internal/driftengine/capture.go:70` calls `AcquireWriteLockWithin` before `runStep`.
  - `runStep` takes the slot at `internal/driftengine/step*.go:266` with `run.Pool.Acquire`, inside the held lock.
  - `internal/driftcli/cli.go:343` still wires `lym drift capture` to `startMainCapture`.
- **Effect:** a main capture can hold the lock while it waits for a slot. That breaks the bound as written.
- **Reach:** narrow. The main capture needs HEAD at the branch point, and checks start only after Drift-Await. Either the bound should name current captures only, or D7 should say the main capture also takes its slot before the lock.

One item outside my lens: in the D9 judge-guard way forward, `git checkout <head> -- <path>` does not undo a moved HEAD, which the same Stuck covers. It also does not remove untracked paths the judge added. That is for the fix-way-forward lens.

Tool call made:
- Grepped `driftengine`'s slot acquisition, `runproc`'s `EndRole` doc and the `lym drift capture` verb. My context had none of them, and I needed them to test the slot/lock order claim and the `EndRole` claim. `EndRole` holds: it ends only the live processes registered under a role.

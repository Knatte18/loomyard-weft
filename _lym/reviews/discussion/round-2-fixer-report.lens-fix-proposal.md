```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1–F9), looking for concrete cases where the proposed change is wrong or breaks something else in the artifact.

Proposals checked: F1 (in-memory migration option, plus the criterion), F2 (whole rebuild starts from an empty map), F3 (registration failure exits; no pass while a live child runs; Bound narrowed), F4 (retake without waiting), F5, F6, F7, F8, F9 (the stamp set on every write that moves `Commit`).

**MEDIUM: F1's chosen option names a step that can't be carried out at the place it names.**
- Changed text: D8 says "beam's `LoadConfig` applies the same `Migrate` to each layer in memory before `configengine.Load` drops the retired key".
- Evidence:
  - `LoadConfig` is `internal/dobbyengine/config.go:78`, and it only calls `configengine.Load`.
  - The layers are merged into `stack.Doc` inside `configengine.Load`.
  - `warnUnknownKeys(stack.Layers, spec)` and then `DropRetired(stack.Doc, spec)` run there too (`internal/configengine/config.go:270-274`).
  - So no caller of `Load` can rewrite one layer before the merge and drop.
- Also, `configreg.Module.Migrate` is `func(existing []byte) ([]byte, []string, error)` over a file's bytes (`internal/configreg/configreg.go:79`), and its returned lines are meant to be logged at Warn (`runsettings.go:46`). Running it on every load would repeat that Warn at every dobby tick until the walk runs.
- What the decision needs: a per-layer migrate hook inside `configengine.Load`, applied before the merge and before `DropRetired`. The record should say that, and say which logging applies.
- The reviewer's other option avoids this: have the `--upkeep` verb and the dobby's upkeep step run `hubreconcile.Ensure` before reading the key.

**LOW: F3 leaves a restart loop when registration fails persistently.**
- Changed text: "A `--upkeep` call whose own registration fails exits before it plans. The dobby starts no pass while a pass it started is still its live child."
- The case:
  1. A runproc directory that can't be written (permissions, full disk) fails the same way every time.
  2. The pass exits before planning, so it writes no exit record and is no longer a live child.
  3. D8's start condition (no exit record, a HEAD change, or queued work) is unchanged, so the dobby starts another pass on every tick, and each one exits at once.
- Neither the record nor the new criterion says how this is surfaced or rate-limited. A once-per-episode log or notice, as the dobby's re-exec failure uses (`internal/dobbyengine/doc.go`), would bound it.

**LOW: F4's "waits under this rule again" stalls the whole pass behind one hand call.**
- Changed text: "when the lock is held, it releases the slot and waits under this rule again."
- The case:
  1. A hand call naming P holds P's build lock for P's whole rebuild.
  2. The yielded pass wakes each second, takes a slot, fails the lock and releases the slot.
  3. Meanwhile every other queued package waits, though they need no lock the hand call holds.
  4. The hand call rebuilds P whole anyway (D9), so the pass's retake of P is wasted once it gets the lock.
- D1 already handles a held lock by skipping the map and planning it again once the lock frees, counting it as queued work. The retake text breaks that pattern; skipping P and moving to the next job would fit it.
- On a busy hub, the take-and-release each second also briefly takes a free slot from a gate arriving at that moment.

No breakage found for F2, F5, F6, F7, F8 or F9:
- **F2:** a discarded first round or a failed build writes nothing, so the old map stays at its old `Commit` and the next plan misses again.
- **F6:** D4's own-file drop under Current agrees with the new text.
- **F9:** under Current and Affected the stamp already matches the current one, since a differing stamp is a miss, so setting it changes nothing there.

Tool calls:
- One Bash call that grepped `configreg` for `Migrate`, found `LoadConfig`, and printed `configengine/config.go:255-290`. My context lacked `Migrate`'s signature and where `Load` merges layers, which F1's feasibility turns on.

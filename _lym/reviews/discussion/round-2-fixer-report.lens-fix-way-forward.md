```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every refusal, exit, wait and next step the fix added can be reached and leads to progress.

1. **MEDIUM.** Changed text: decision-record.md D8, sixth bullet: "Before any walk has run, beam's `LoadConfig` applies the same `Migrate` to each layer in memory before `configengine.Load` drops the retired key".
   Summary: as written, beam's `LoadConfig` cannot do this step, because it never sees the layers before the retired key is dropped.
   Evidence:
   - `internal/dobbyengine/config.go:79`: `LoadConfig` calls `configengine.Load(dirs, ConfigSpec())` and gets back only the merged, resolved bytes.
   - `internal/configengine/config.go:185-274`: `Load` reads the layers itself and calls `DropRetired(stack.Doc, spec)` on the merged stack. The caller has no point at which it can rewrite a single layer.
   - Closing F1's window needs a per-layer migrate hook on `configengine.Spec`, applied in `Load` before `DropRetired`, and the record names no such hook. A build task working only from this text would be stuck.
   - `configreg.Module.Migrate` (`internal/configreg/configreg.go:79`) belongs to the registry, which imports `dobbyengine`, so `dobbyengine` cannot call it through configreg without an import cycle. The migrate function has to live in `dobbyengine` and be wired into the spec.

2. **LOW.** Changed text: D8, second bullet: "A `--upkeep` call whose own registration fails exits before it plans."
   Summary: the exit gives no way forward. It writes no exit record and reports nothing, so the dobby restarts the call on every tick while registration keeps failing.
   Evidence: D8's start condition (no exit record, HEAD differs from the planned HEAD, or queued work) still holds after such an exit, because the call "exits before it plans". Its only bound is "the dobby starts no pass while a pass it started is still its live child", and that lapses as soon as the call exits. Nothing says the failure is logged or posted, so an operator never learns why upkeep makes no progress. Only a runproc directory write failure gets here, which is why this is LOW.

3. **No finding.** D9's retake ("tries that lock again without waiting; when the lock is held, it releases the slot and waits under this rule again") is reachable and makes progress. The hand call keeps the lock only for its own job, the wait wakes at most once a second with backoff, and HEAD is read on each wake.

Tool calls:
- Read `internal/configengine/config.go:255-290` and grepped `configreg` for `Migrate`, to find where `Load` drops retired keys and what shape `Migrate` has. My context did not have `configengine`'s internals.
- Grepped `dobbyengine.LoadConfig` and `configengine.Load`'s signature, to see whether beam's `LoadConfig` can reach the layers before `DropRetired`. My context did not have `LoadConfig`'s body.

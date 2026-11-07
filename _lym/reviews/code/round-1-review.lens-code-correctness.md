```yaml
lens: code-correctness
outcome: reported
```

**Correctness list (lens: code-correctness)**

- `validateObjectEntry`
  - Text `null`, `[..]`, `{} x` and `{"a":1}}` are refused.
  - An object target beside a text target is refused.
  - A text-only target list passes. Correct.
- `decodeObject`
  - Empty string, `null` and trailing garbage return false.
  - Trailing whitespace passes. Correct.
- `mergeJSON`
  - Object+object recurses, list+list dedups, and a scalar or a type change takes `over`.
  - Inputs are never mutated. Correct.
- `foldObjects`
  - Entries fold in order.
  - Wrong: an entry with invalid text still sets `held=true`. See F1.
- `dropStale`
  - A scalar is removed only while the file still holds heddle's value.
  - A scalar still in `next` is kept and then overwritten by the merge.
  - A container the removal empties is deleted.
  - A user key such as `allow` is kept.
  - A key whose type changed is kept. A nil `previous` is a no-op. Correct.
  - Minor: an entry-set empty container (`{"env":{}}`) is never removed (a LOW cosmetic leak).
- `readObjectFile` and `readRecord`
  - A missing file gives `{}`/false, a blank file `{}`/true, and a non-object nil/true. An invalid record is treated as no record. Correct.
- `objectRefusal`
  - It checks, in order: blocked staging directory, file off the exclude list, `.heddle` off the list, then file holds no object. Correct.
  - With no entry set (`!held`) a refusal records nothing in `Unwritten`. This is intended.
- `renderObjectTarget`, held path
  - With an absent file it creates `.claude/` and writes the file, then the record. Correct.
  - Wrong on failure: see F2.
- `renderObjectTarget`, `!held` path
  - It drops the heddle-set keys, removes the file if empty, and otherwise writes the remainder. Correct.
  - Wrong on failure: see F2.
- `Render` and `Show`
  - Dispatch on `JSONObject` is right. `Show` returns "" with no entry.
  - The staging name changed to `target.Name`. That is consistent with the updated leftover-temp test.
- `writeRendered`
  - It creates the path's directory. Correct.
- `SetOverride`
  - It validates against the shipped entry's targets. Correct.
- `AppendExcludePaths`
  - Key unset, path already listed, no mapping root and parse error each return as documented.
  - A list lacking the path gains it, with the line comment kept.
  - `warp_exclude:` (null) or `""` gains the path, which is acceptable.
  - A second run is a no-op. Correct.
  - Minor: the re-encode reindents the whole file to 2 spaces (a NIT).

**Findings**

- **F1 — MEDIUM, design.** `internal/heddleengine/settings.go:83-92`, `:263`.
  - `held` is set before decoding. When every entry for the target is invalid (a hand-edited store or override), `folded` is `{}` and `held` is true.
  - The render then runs `dropStale` with an empty `next`. That strips every key heddle set last render, and the record is overwritten with `{}`.
  - Such an entry is warned about and meant to be "left out", but its settings are taken out instead of kept.
  - Fix: count `held` only for entries that decode. Or, when any named entry is invalid, skip the take-out and keep the previous record.
  - Loading does validate most paths (`shipped.go:47`, `store.go:271`, `replayer.go:146,163`), so this is reachable only through hand edits and override files.
- **F2 — LOW, design.** `internal/heddleengine/settings.go:266-268` and `:283-297`. The order of record update against file write is unsafe on a failure.
  - `!held` branch: the record is removed before the file is written or removed. If the write or removal then fails, the stale keys stay in the file with no record, and later renders never take them out.
  - Held branch: the file is written before the record. If the record write fails, new keys are in the file but absent from the record, so they are never taken out later.
  - Fix: in the `!held` branch, remove the record after the file write or removal succeeds. In the held branch, accept the order or write the record first with the union of old and new.

**Tool calls**
1. `grep ValidateEntry/validateObjectEntry` in `internal`: to see which load, replay and set paths validate entry text, since the manifest lacked `shipped.go` and `replayer.go`.

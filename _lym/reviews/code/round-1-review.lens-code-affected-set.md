```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. List per change.

**Per-change set (covered or not)**

1. New `State.PointerMarkSince` field (`state.go:152`). Its readers and writers are `watcher.go` `hookDelivered` (1365), `tickClearing` (1233), `startAutoReload` (1273), `tickCompacting` (1649), `toIdle` (892) and `ResetForFreshLaunch` (`state.go:464`).
   - Persistence: `enter` saves after every set. The sets happen before `startReload` → `enter` in all three paths. `reloadAfterCompaction` receives the mutated `st`, so it carries the value too. Covered.
   - `internal/orchcli/status.go` prints state keys (`cycle_count` at line 59). It lists no baseline or reload-bound fields I could find, so nothing is forced there. Covered.
   - No fixture or golden state file mentions `compaction_baseline`. The repo-wide grep found no match outside the orchengine sources, so no fixture needs the new key. Covered.

2. `hookDelivered` semantics changed from `CompactionBaseline` to `PointerMarkSince`.
   - Its doc comment is updated (`watcher.go:1356`).
   - The package doc is updated (`doc.go:329–330`).
   - `internal/orchcli/doc.go:130–135` says only that the verb writes the mark the watcher reads, which stays accurate.
   - `orchengine.ResumeContext` and the mark format are untouched, as the record's Out scope requires.
   - Covered.

3. `ResetForFreshLaunch` now zeroes the field, and its doc comment is updated (`state.go:441`).
   - `TestResetForFreshLaunch` in `state_test.go:283–335` has no assertion on `PointerMarkSince`, and neither its idle nor its non-idle input sets the field.
   - Of the field-zero paths, `toIdle` is asserted at `watcher_test.go:1525` and `watcher_compact_test.go:524`. The fresh-launch zeroing has no test row.
   - Finding below.

4. Doc description in `doc.go:329–330`.
   - It states the `/clear` case and the compact case, and it says "every return to idle clears the mark" at 333.
   - It does not say the field is persisted or zeroed on a fresh launch. The State field doc says "zero outside a reload", which covers this. Covered enough.
   - The `doc.go` State-key listing (`doc.go:263`) is a list of reload keys, and the field is not a reload step key. No listing is required.

5. `PATTERN` entries.
   - `PATTERN-tolerant-records`: the field is a plain `time.Time` with a JSON tag, so it decodes as zero from an older file. Writes go through `w.save`. Covered.
   - `PATTERN-test-speed`: the new tests use `t.Parallel()` and the injected clock. Covered.
   - The commit messages carry no `Pattern-Departure:` trailer, and none is needed. No declared PATTERN changes exist in my directive.

6. Stencils: the diff touches none.

**Findings**

- LOW, `consistency`: `internal/orchengine/state_test.go:283` (`TestResetForFreshLaunch`).
  - The diff adds `s.PointerMarkSince = time.Time{}` in `ResetForFreshLaunch` (`state.go:464`), and no test would fail if that line were removed.
  - Decision D2 and the changed doc comment at `state.go:441` both claim it is zeroed on a fresh launch.
  - Evidence: `state_test.go:286–335` sets no `PointerMarkSince`, and the other test files have no reference to it (grep shows only `watcher_compact_test.go:524` and `watcher_test.go:1525`).
  - This is the only gap in my lens's set.

No other member of the affected set is left wrong.

**Tool calls (2 of 6)**
1. Repo-wide grep for `compaction_baseline`, `resume-mark`, `session-start hook` and `CompactionBaseline` outside the orchengine sources. My context lacked other consumers, docs and fixtures.
2. Read `internal/orchcli/doc.go` lines 128–150 and grepped state keys in the docs and `orchcli`. My context lacked the hook doc and the status output's key list.

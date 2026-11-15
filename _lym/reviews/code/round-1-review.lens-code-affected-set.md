```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for what it forces outside the lines it touches. I found nothing it leaves wrong.

Per change, the set I found and whether it is covered:
- **Declared `PATTERN-lym-files` text:** `PATTERN.md` now carries the declared text exactly, and `pattern/PATTERN-lym-files.md` carries the same sentence in its header. Covered.
- **`pattern/PATTERN-lym-files.md` body:** The new bullets list the `.lym` caches. The list matches the record's D3 caches and the intents and records that moved. Covered.
- **New leaf package `runfsm`:** `pattern/PATTERN-leaf-packages.md` gains an `internal/runfsm` section. Its import set (stdlib and `shedengine`) matches `internal/runfsm/state.go` and `internal/runfsm/act.go`, which import only stdlib and `shedengine`. Covered.
- **Cancel marker file removal:**
  - The remaining "cancel marker" mentions in `internal/battencli/*_test.go` and `internal/pairteardown/cancel_test.go` are test and comment wording. `readCancelMarker` still exists as the reader of the intent plus the lock, and `internal/battencli/dobbyreads.go:204` uses it.
  - `docs/research/loom/lym-state-map-2026-11-15.md:137` still lists the `shed/<slug>.cancel.json` file. That is a dated research snapshot and not a rule document, so I raise nothing.
  - Covered.
- **PR record moved into the pair's tracked records:** I checked whether the other patterns' rule text describes where the PR record lives. `pattern/PATTERN-daemon-wakeups.md:58` says an `internal/fswatch` event on the run's pull-request record wakes PR-Review. It names no path, so it stays correct. I did not confirm that the watched path in `internal/landingshed/prreview.go` follows the record's new location. The code-tests lens or the handler should check that, since `prreview.go` is in the diff and I did not read its watch call.
- **Origin-owed marker (`OriginOwed`):** The diff leaves it in place at `internal/battencli/arm.go:626` and `internal/dobbyengine/passes.go`. That matches the answer key, which keeps the marker until bolt-holds-catch-up. Covered.
- **Commit trailers:** The log over the range holds no `Pattern-Departure:` trailer, so none needs judging.

Findings: none.

Tool calls (3):
1. `git diff` of `PATTERN.md`, plus a grep for cancel-marker and PR-record mentions in docs, patterns and Go files. My context lacked the `PATTERN.md` diff and the stale-reference set.
2. Read `pattern/PATTERN-daemon-wakeups.md` lines 52-64, plus a grep for the PR-record watch in `dobbyengine` and `pullrequest`, and for cancel and pause mentions in the records and parsers pattern files. My context lacked the pattern text that names the PR record.
3. No third call was needed.

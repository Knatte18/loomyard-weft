```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I checked everything outside the touched lines that the diff's changes force or affect, and found nothing the diff leaves wrong.

**Per change, the set found:**

1. **`watchedSession` gains a `running *sync.WaitGroup` field** (`internal/reedcli/watchdog.go:204-209`).
   - `enterSession`'s three error returns build `watchedSession{}` with a nil `running` (`watchdog.go:338,343,348`). Every caller drops the value on error (`watchdog.go:606-612`, `entersession_integration_test.go:45-48`, `watchdog_integration_test.go:511-514`), so nothing calls `Wait` on nil. Covered.
   - The `watchdog_test.go:46-73` literals build `watchedSession` without `running`. `planSessionDiff` reads only the map keys (`watchdog.go:215-229`), so they are unaffected. Covered.
   - The daemon's cancel sites (`watchdog.go:482,596,621`) call only `cancel` and never read `running`. That matches the summary's design decision. Covered.
   - The doc comments on `runWatchdogLoop` (`watchdog.go:432-464`) and `enterSession` (`watchdog.go:320-334`) describe cancelling, not waiting, so they stay accurate. Covered.
   - No markdown doc, spec or stencil names `watchedSession`, `enterSession` or `stopWatchedSession`; the grep outside `_lym/reviews`, `.lym` and landing found none. Covered.
2. **`enterSession` starts its goroutines through `running.Go`** (`watchdog.go:354-371`).
   - `running.Go` needs Go 1.25 or later; `go.mod` declares `go 1.26`. Covered.
   - Behaviour is unchanged apart from being awaitable. The callers are the daemon loop and the two tests. Covered.
3. **New test helper `stopWatchedSession`** (`entersession_integration_test.go:58-74`), used by both tests' cleanups.
   - Under PATTERN-test-only-code it sits in a `_test.go` file. Covered.
   - Under PATTERN-test-speed, its 30s limit is a deadline local to the test, not a production duration the test waits out. Covered.
   - Both files are in the same package with no differing build tag, so the helper is visible to `watchdog_integration_test.go:515`. Covered.
4. **Commit message and departures** (`git log 8641cd504..HEAD`, commit `9af002af6`).
   - The `Root-Cause:` trailer is present and names the defect: no way to await the goroutines' exit.
   - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed. No assertion was widened, since only cleanups changed.
5. **Declared PATTERN changes:** the directive declares none, so there is nothing to land.
6. **Stencil wording:** the diff touches no stencil, so this does not apply.

Tool calls:
- One grep over `*.md` for the three symbol names and over `internal/reedcli/*.go` for `watchedSession{` literals. My context lacked the doc, spec and stencil set, and the full list of literal construction sites.

```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test that pins it.

**Behaviour → pinning test.** All of them are in the single subtest `activity_test.go:290`, "a running record whose waiter is gone logs once per episode until it reads live".

- **Warn once, Debug on repeat, Info when the waiter is live again.** Pinned at `activity_test.go:305-318`. It reads three times while waiterless, then rewrites the record as live. `requireFailureEpisode` (`fakes_test.go:95-115`) asserts 1 Warn, 2 Debug lines stamped `failed_ticks=`, and 1 Info.
- **A running record with a dead pid is still skipped.** Pinned at `:305-309`, which asserts zero readings. The live-again read returns 1 reading at `:313`.
- **A record with no pid logs nothing.** Pinned by `run-b` (running, pid 0), via the final check that `run-b` never appears in the log (`:346-350`). This holds, but only through that log check.
- **A finished record logs nothing.** Pinned by `run-c` (done, pid 4343), through the same log check.
- **Episode ends when the record turns terminal.** Pinned at `:321-330`, which expects the second Info.
- **Episode ends when the record vanishes (the `!found` path).** Pinned at `:331-341`, which expects the third Info and no further Info on the repeat reads.
- **Separate episode set from `unreadableFiles`.** No dedicated test. The three-read episode in the first phase would emit 3 Warns if the key were shared, so it is covered indirectly.
- **`doc.go` paragraph.** Static prose, exempt from coverage under the rubric.

**Test economy.**
- The test duplicates nothing. The two existing episode subtests (`:246`, `:263`) cover unreadable events, a different path. They also use `requireFailureEpisode`, so the new test reuses their assertions.
- It is not a per-helper test: it drives `ReadAgentActivity`, the public surface, and `noteWaiterlessRun` has no test of its own.
- No review-fix test appears in the diff, and no assertion targets stencil prose.

**Findings.** None.

**Observation (not a finding).** The check at `:342-350` uses `log`, a snapshot taken at `:311` before the later phases. It still excludes `run-b` and `run-c` for those later phases only because it was taken early. This is harmless, since those runs never change, and I'm not raising it.

**Tool calls (1 of 6):** I ran one `grep` over `activity_test.go` and `fakes_test.go` for the `requireFailureEpisode` and `countLevelLines` helpers and the subtest layout. My context lacked those helpers' bodies.

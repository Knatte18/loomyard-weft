```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour, the test that pins it**

1. In `reedengine.downCause`, a dead pane gives `pane-dead`. Pinned by `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`, strand `dead` (`internal/reedengine/lifecycle_test.go`, the new `wantDown` map).
2. A pane in another session gives `pane-elsewhere`. Pinned by the same test, strand `gone`.
3. A pane that no listing carries gives `pane-unlisted`. Pinned by the same test, strand `nowhere`.
4. A start shell past the grace gives `start-shell`. Pinned by the same test, strand `shellonly`.
5. A live strand carries an empty cause. Pinned by the same test: strands `w`, `alive` and `engine` are absent from `wantDown`, so they are expected to be `""`.
6. A strand bound to no pane (empty `PaneID`) gives `pane-unlisted`. None.
7. A failed server-wide lookup gives `pane-unlisted` instead of `pane-elsewhere`. None: the failed-lookup loop asserts only `Elsewhere`.
8. On Windows, where the shell rule is skipped, no strand gets `start-shell`. None. This depends on the platform, so I do not file it.
9. In `orchroute.readingFrom` and `ReadingOf`, the cases no strand recorded, live, reed's cause passed through, strand not in reed's state, no reed session and other error are each pinned by a row of `TestReadingFrom` (`internal/orchroute/live_test.go`).
10. `orchroute.Live` logs each down reading, at Debug for no strand recorded and at Info otherwise. None.
11. The `strand_down` field of `lym orch status`:
    - `no-strand-recorded`: pinned by the `TestStatus_Envelope` row "unknown tokens are null".
    - `pane-elsewhere`: pinned by the row "a pane in another session is named".
    - `strand-not-in-reed-state`: pinned by the new row "a strand missing from reed's state reads down with that cause".
    - A live strand's empty `strand_down`: no `orchcli` row asserts it. The orchroute row "A tracked live strand" covers `ReadingOf`, which the verb calls directly, so I file nothing.
12. The help text and doc comments are static prose and exempt.
13. The changed `orchcli` fixture: the row "a pane in another session is named" now sets `Live: false` and asserts `strand_live: false`. That is stricter than before, not looser.

**Findings**

- **LOW, design.** Location: `internal/reedengine/lifecycle_test.go`, the failed-lookup loop in `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`. When the server-wide lookup fails, `downCause` should read strand `gone` as `pane-unlisted`. No assertion pins that: the loop checks only `s.Elsewhere`, so a cause that stayed `pane-elsewhere` after a failed lookup would pass. Evidence: `internal/reedengine/apply.go`, the `elsewhere != ""` branch of `downCause`, and `internal/reedengine/ownwindow.go`, where `strandsElsewhereLocked` returns nil on a lookup error. Fix: in the failed loop, also assert `s.Down == DownPaneUnlisted` for `gone`.
- **LOW, design.** Location: `internal/reedengine/lifecycle_test.go`, the same test's fixture. The doc of `DownPaneUnlisted` promises the cause for a strand bound to no pane, and no test pins it. The fixture has no strand with an empty `PaneID`. Evidence: `internal/reedengine/apply.go`, the doc comment of `DownPaneUnlisted` ("or that is bound to no pane"). Fix: add a strand with `PaneID: ""` to the fixture and expect `DownPaneUnlisted`.
- **LOW, design.** Location: `internal/orchroute/live.go`, the switch after `readingFrom` in `Live`. No test pins the log levels: Debug for `no-strand-recorded`, Info with the cause for every other down reading. The record asks for that log line. Evidence: `internal/orchroute/live.go`, the `logger.Debug` and `logger.Info` calls; `TestReadingFrom` tests only `readingFrom`. The handler may reject this finding, since a log line is read by humans and the testing rules exempt output only a human reads.

Tool calls: none.

---
verdict: APPROVED
---

No findings.

The range is `git diff $(git merge-base main HEAD)..HEAD`, since `_lyx/loom/rework/` does not exist and `product.parent` in `_lyx/shed/logging-cleanup/status.json` is `main`.
`lyx gate test ./...` and `lyx gate test --tags integration ./...` both pass with no failing package.

Focus questions, each settled against the diff:

- Cards 1 and 2: `logger.DefaultOutput` returns `io.Discard` under `testing.Testing()`, and every `SetOutput` restore in the tests, including `logcapture.Capture`, uses it.
  The one test helper that needs the real stderr (`gatecli`'s `runGateHelper`) sets it explicitly.
  `Episode` warns on the first failure, logs Debug with `failed_ticks` while it repeats, and logs Info at recovery and resets.
  `EpisodeSet` guards its map with a mutex.
  `PATTERN.md` carries `PATTERN-log-levels`, and the four shortened entries match the card.
- Cards 3, 4, 9, 11 to 15: each guarded site in the D11 table wraps its line in an `Episode` or `EpisodeSet` with the keying the row states (per strand, pane, entry, path or message), and calls `Recover` on the success path.
  Lines that log and return no error carry their own guard, such as `ReadAgentWaits` and the wait markers keyed by path.
- Cards 8, 9, 11, 14: the demotions are in place at the named lines.
  Publish-retry and resume attempts use `logAttempt` (Info on the first attempt, Debug on retries); `childGoto` and `Spawn` log at Debug; `releaseLock` and the idle release log at Info; `session state at notice` logs at Debug.
  No retry path logs at Info after its first attempt.
- Card 5: the watchdog stats its binary on each timer tick, retires through the deferred teardown, releases the lock and respawns once.
  The successor takes a fresh baseline, so there is no loop; a failed stat counts as unchanged.
  Lock contention logs at Debug.
- Cards 10 and 12: `IdleProbe.Draft`, `SessionBusyError.Draft`, `ErrForeignBoxText` and `IsDraftHold` carry the draft kind apart from the text, and `heldReason` and `heldTyping` substitute `draftHoldReason`, so no orch log line or `State.Stuck` holds draft text.
- Cards 16 to 19: teardown archives the trace and verify logs before removal and never fails on it, prunes at 30 days, and writes under `fabricengine.LogArchiveDir`.
  `ready` queues a notice on every landing, and the orch stencil routes every `landed` notice to After landing with the dropped-notice checks.
  `pairteardown`'s `os.RemoveAll` is outside `PATTERN-fabric-destruction-chokepoint`, which is scoped to `package fabricengine`.
- Test economy: each new top-level test is named in its card, and the per-package counts stay within `cmd/lyx/testdata/test-budget.yaml`, which `cmd/lyx/testbudget_test.go` enforces and which passed.
  No new test duplicates the coverage of an existing one.
- Docs: the owning `doc.go` files (`battenshed`, `orchengine`, `pairteardown`, `reedcli`, `reedengine`, `shuttleengine`) and `docs/overview.md` change with the behavior.

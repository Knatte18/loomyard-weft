```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the changed code of fixes F1–F6 for correctness, and whether each added test fails with its defect in place.

**MEDIUM — the F3 test passes without the F3 fix**
- Changed text: the new directory assertion in `internal/beamcli/dobbydeps_integration_test.go`, placed after `primeDeps.Tick(...)`.
- Evidence: `Carrier.Tick` calls `takeUp`, which calls `updateRequest`, which calls `state.UpdateJSON`. `UpdateJSON` runs `os.MkdirAll(dir, 0o755)` at `internal/state/state.go:150`, before it reads the file.
- So the `Tick` call, even with no request present, creates the hibernate directory. The `os.Stat` that follows then succeeds with or without the `MkdirAll` added to `NewDobbyCarrier`.
- Fix: stat the directory straight after `hubDobbyDeps(prime, primeGeom)` returns, before `primeDeps.Tick` runs.

The rest holds:
- **F1:** the table check and the reverse loop both fail if the literal list drifts from the engine's `Terminal()`.
- **F2:** with the phase check removed, the done, failed and cancelled rows return true and fail. The dead-carrier row (own pid, wrong start time) is deterministic.
- **F4:** the reason keeps the substring `ended in phase <phase>` that the existing assertions use. The new notice parts fail without the change.
- **F5:**
  - `wakeWatch` runs after `running` is cleared, so the following `Tick` can take up the new request.
  - A terminal or missing request causes no write loop, since `takeUp` returns `errUnchanged` for a terminal request.
  - The mtime wait fails if `wakeWatch` is removed. Nothing else writes the file after the stale `Chtimes`, because the old sequence's `stillOurs` check returns false first.

Calls made:
- `grep UpdateJSON in internal/state/state.go`: to see whether `UpdateJSON` creates the directory, since the F3 test calls `Tick` first.

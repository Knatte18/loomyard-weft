# Round 1 fixer-report

## Changes

- **F1 (fixed as proposed, with the bound read per role):** `gateProcessLive` in `internal/battencli/wire.go` takes `now` and counts a fixer record only while `now - RegisteredAt` is within the fixer's own timeout; a `runproc.RoleGate` record still always counts.
  The new `fixerTimeout` reads the bound from the config the fixer is spawned with: `gateslot` `FixerTimeout()` for a gate fixer, webster's `RecoveryTimeoutMin` for a verify fixer.
  An absent gate config leaves a gate fixer unbounded, so it counts, as the review's other readers of that config tolerate `ErrConfigAbsent`.
  `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` gains rows for a gate fixer and a verify fixer registered 24 hours ago (no wait) and a gate runner of the same age (a wait), and seeds the gate config from `gateslot.ConfigTemplate()`.
  `_lym/landing/summary.md` now states the age bound instead of the unenforced claim; the earlier commit message made no such claim, so nothing there needed correcting.
- **F2 (fixed as proposed):** the same test registers a gate runner under `another-slug` before any same-slug registration and expects waitLive false with a nil error.
- **F3 (fixed differently):** `TestRun_Wait_Classification` asserts, outside the cleanup switch, that the run-end stamp exists and is not older than `result.EndedAt` minus one second.
  The assertion is skipped for rows that want an error: the two mechanism-failure rows return before `finalize` and leave no stamp, which the first gate run showed.
  The existing `ReadNewestRunEnd` assertion stays in the cleaned-up done case.
- The consistency lens's doc findings are fixed in `wire.go`'s `readAgentActivity` comment, the test's doc comment and the summary's Tests section.

Commits: `shuttleengine: fix F3`, `battencli: fix F1, F2` and `battencli: fix F1 — name the fixer age bound`.
F1 and F2 share one test hunk, so they share a commit.
`lym gate test ./internal/battencli ./internal/shuttleengine` passed (tier0, 0 failed) after the code changes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the summary's Tests line was stale — fixed as proposed.
- fix-consistency, LOW: the test's doc comment read unconditional — fixed as proposed.
- fix-consistency, LOW: `readAgentActivity`'s doc comment omitted the age bound — fixed as proposed.
- fix-consistency, LOW: the earlier commit message — no action, it holds no claim to correct.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

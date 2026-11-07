```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set** — what the diff forces outside the lines it touches. Per-change list:

| Change | Set found | Covered? |
|---|---|---|
| `newStaleHub(t, slug)` is now `newStaleHub(t)` | 9 call sites at `ensure_integration_test.go` lines 512, 776, 808, 865, 1055, 1171, 1208, 1265, 1284 and 1563, all converted. No other file in the repo calls it. The only other mentions are the research report `docs/research/speed/tier1-test-revision-2026-11-06/report.md:121,200`, a point-in-time measurement (the prompt's measurement-report exception). | covered |
| `seedRetiredKey`, `replaceOnce` and `commitPreMigrationFiles` take `testing.TB` | These are the only helpers the Build calls. `commitPreMigrationFiles` calls `replaceOnce`, which is converted. `gitkit.Git` and `gitkit.CommitFile` already accept TB (the Build passes `tb` to them). Other callers pass `*testing.T`, which satisfies TB. | covered |
| `arrangeUnpushedHubWide(t, h)` takes the hub as a parameter | Two callers, both updated. The unused `hubforge` import is not an issue, since `hubforge` is still used. | covered |
| `TestEnsure_HeldLockTimesOutWithoutStamping` moved to `stamp_test.go` | No docs, gate keys or redundancy reports name it. `errors`, `lock`, `time`, `os` and `filepath` are all still used in `ensure_integration_test.go`. `stamp_test.go` is in the internal package. It has no TestMain, but the same test binary holds `testmain_integration_test.go`. It spawns nothing and uses no hubforge, so it satisfies `PATTERN-test-speed`'s tier-0 purity. `ensure` (`walk.go:43-61`) reaches the lock with an absent stamp, so the tier-0 geometry is valid. | covered |
| `stamp_test.go` file header | Updated to name the held-lock walk. | covered |
| `ensure_integration_test.go` file header, lines 1-3 | It still says the file tests "the lock". The only lock coverage left there is the held-lock subtest of `TestEnsureOrWarn_WarnsWithTheWayForward`, so this is mostly still true. | see LOW below |
| `NewHub` sites converted to `CopyHub` | `grep` over `internal/hubreconcile` shows no `hubforge.NewHub` left. The `CopyHub` sites are in `ensure_integration_test.go` and `reinterpret_integration_test.go`. The latter was already on `CopyHub` and is not in the diff. D5 asks that a kept `NewHub` site name its reason, and there are none. | covered |
| Process-global seam ordering | In `TestEnsureOrWarn_WarnsWithTheWayForward`, `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` and `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`, the copies come before `logcapture.Capture` and `boardkit.InstallReplay`. In the subtest of the last one, `logcapture.Capture` runs before `arrangeUnpushedHubWide`, but that call takes no copy, so the order is fine. | covered |
| `PATTERN` declared changes | The directive declares none for this diff. The `pattern/PATTERN-test-isolation.md` registered-template text was landed by the earlier hubforge commit and is not part of this range. | n/a |
| Commit messages and `Pattern-Departure:` trailers | No trailers, and none are needed. The commit message matches the diff. | covered |

Findings:

1. **LOW / consistency** — `internal/hubreconcile/ensure_integration_test.go:1-3`. The file header says the file "also tests the lock", but the dedicated held-lock test left this file. Only the held-lock subtest of `TestEnsureOrWarn_WarnsWithTheWayForward` (line 1155) still touches the lock. This is borderline and arguably still accurate, so I'd raise it only as a nit.

I found no other member of the affected set left wrong.

Tool calls (1 of 6):
1. `grep -rn "HeldLockTimesOut|newStaleHub|arrangeUnpushedHubWide"` across the repo, excluding the two files and `_lym`/`.lym`. My context only held the two test files, so this checked for docs and other consumers.

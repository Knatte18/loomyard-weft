---
round: 2
focus:
  - "cmd/lym/fslinkonly_test.go, `linkCallFailures` and its `TestFslinkOnly_FixtureTrees` rows: settle whether a file that binds `os` by dot import, by alias or by blank import is handled so that no `os.Symlink` or `os.Link` call outside `internal/fslink` can pass the scan unseen, and whether the fixture rows pin each binding."
  - "cmd/lym/interactivespawn_test.go, `headlessSpawnFailures` and its fixture loop: settle whether the allowlist is consulted only for a file that has findings, so that an entry for a file with no remaining finding is reported by the allowlist's stale check, and whether a fixture row built with a non-nil allowlist pins both admission and staleness."
---

Both entries target the two gating areas of round 1, judged against decision record D2 and D4 and the PATTERN entries `PATTERN-interactive-spawn` and `PATTERN-fs-links`.

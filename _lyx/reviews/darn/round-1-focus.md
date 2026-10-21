---
round: 1
focus:
  - "tools/deploy/main.go `otherLyxBinaries`: does it still report a stale lyx copy that is a symlink to a different file, and does seeding `seenFiles` with the resolved production binary hide a real second copy or report the production binary through a PATH alias? Settle against the board's 'does not report one file twice' expectation."
  - "tools/deploy/main.go `otherLyxBinaries` build-info check: does `buildinfo.ReadFile` identify a lyx built by `deploy` itself (cmd/lyx main package path under `go build` with -ldflags), and does it behave on Windows (`lyx.exe`, `binaryName()` use) the way the old `fileExists` path did?"
  - "tools/deploy/main.go `pushedGate` and `run`: after dropping the `prod` push, is the retained refusal of a HEAD not on origin/main intact, and is any `head`, `prodBranch` or ancestry leftover, in code, comments or the package header, still referring to the removed pointer?"
  - "Scope against the board entry: the diff touches only `tools/deploy`, `CLAUDE.md` and `README.md`; check that every other place that says the script moves or reads `prod` (docs, PATTERN, scripts, `update-plugins.*`) is either updated or is not licensed by the entry."
  - "tools/deploy/main_test.go `TestOtherLyxBinaries_ReportsEachLyxCopyOutsideTheDestDirOnce`: does the test cover the behaviors the diff adds (foreign program skipped, directory-link alias reported once, production alias skipped) without duplicating another test, and does it hold on Windows where a shell-script `lyx` and directory links differ?"
  - "`_lyx/landing/summary.md` claims against the diff, notably the 'pushedGate has no unit test' rationale and the 'each file once' and 'production alias never reports' claims."
---

Seeded from the board entry `deploy-prod-cleanup` and the committed diff over the merge base.

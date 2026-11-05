# Round 1 fixer-report

## Changes

- **F1 (MEDIUM), fixed as proposed**, commit `f894f29c`.
  `internal/shuttleengine/claudeengine/command.go`: the `OpenLaunch` doc comment now says the opened session's `settings.json` holds "creel's hook delivery and the empty attribution and nothing else", and the `writeOpenFiles` comment says it writes the hook delivery over `creelFlagDirPosix` and the empty attribution.
- **F2 (LOW), fixed as proposed, in part**, commit `d467e38a`.
  `internal/shuttleengine/claudeengine/settings_test.go`: the `RunStandalone` and `Opened` rows are gone, and the table is dropped.
  The one remaining case, the run document built with a creel flag directory, is the single-case test `TestBuildSettings_CreditNoSessionInAHub`.
  The `summary.md` correction is deferred, see below.

`lym gate test ./internal/shuttleengine/claudeengine` passed: 752 passed, 0 failed, 5 skipped.

## Self-check

- fix-code-correctness: returned no findings.
- fix-proposal: returned no findings.
- fix-affected-set: returned no findings.
- fix-consistency: LOW, `_lym/landing/summary.md:18` still names `TestSettings_CreditNoSession` and its claim that the opened document had no test of its own content; deferred, the same item as the `summary.md` correction below.

## Disputed

None.

## Deferred

- **F2, the `summary.md` correction (and the fix-consistency LOW on it).**
  The file is `_lym/landing/summary.md`, and my instructions bar touching `_lym`: the loop that spawned this round owns it.
  The line to correct is line 18, which should name `TestBuildSettings_CreditNoSessionInAHub` as pinning the empty attribution in a run document built with a creel flag directory, and drop the claim that the opened document had no test of its own content, since `checkOpenCreelFiles` already asserted its hooks and `attribution`.
  Left to the run's parent.
- **F3 (LOW), commit `16bac6c9`'s Co-Authored-By trailer.**
  The finding needs no file edit and asks for a history rewrite, which this round must not do: it commits per fix and never rewrites or pushes.
  Left to the run's parent for the next time the commit is rewritten.

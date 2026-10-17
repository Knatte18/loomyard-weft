---
verdict: APPROVED
---

# Round 1 review

Range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, no rework round exists), three commits, one per card.
All three cards land as written, and no finding survived consolidation.

## Focus answers

- `Store.Log` (`internal/creelengine/verbs.go`): the id clause keeps an entry whose `ID` equals the filter or whose `Unread` holds it, `From`/`To`/`Since` still apply to every entry, and a zero `filter.ID` short-circuits to true, so it filters nothing.
- `writeRefusal` and `runVerb` (`internal/creelcli/cli.go`): the stderr fallback is correct for each combination of `done` and a prefixed or unprefixed `msg`.
  The lost-answer line goes straight to `cmd.ErrOrStderr()`, never through `writeRefusal`, so no error envelope follows a failed or partial success write.
- Verb bodies set `done` as card 3 lists: post (including `--reply` and returned-undelivered), inject (all results), forward, hold, release, and interrupt only when `Sent`.
  Inbox, log, read, the no-running-turn interrupt and the error-beside-result paths leave it empty.
- `internal/creelcli/cli_test.go`: the new rows extend existing subtests or add the one subtest per behavior the cards name, and `DormantStrand(beta)` keeps the carried-entry fixture free of reminders.
- Docs: both `doc.go` files, S9 of `tools/sandbox/SANDBOX-CREEL-SUITE.md`, the `logCmd` help strings and the doc comments of `Store.Log`, `result`, `writeRefusal` and `runVerb` state the new behavior.

Commit messages carry no `Pattern-Departure:` trailer and none is needed.
The directive's declared PATTERN changes list is empty.

## Rejected

- LOW `releaseCmd` sets `done` even when `resp.Released` is false (lens:code-correctness): card 3 lists release with forward and hold as setting `done` unconditionally, and only interrupt is gated on `Sent`.
- NIT the prefix-once rows run in a plain loop, not `t.Run` (lens:code-tests): each failure message names the row, the card asks for rows in one subtest, and the rows share one rig.

---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: ruling
    reason: The parent accepted the top-level keyword widening in decision notice #2155 and asked for no code change; the target already holds it, so nothing was changed.
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

- F1 (`internal/shellguard/rows.go`): added `shellScope`, with the three scopes top level, single-quoted script and double-quoted script, and `commandRowPattern`, which renders one alternative per scope.
  Each scope carries its own argument word, word class, go arguments run and mutation scan, so a scan stops at the script's closing quote.
  The sed, perl, tee, sleep, headless claude, adopt, publish and go rows are built through it.
  The old unscoped `shellArgumentPattern`, `goWordPattern`, `goSubcommandPattern`, `goArgumentsPattern` and `ghMutationScanPattern` are gone.
  Beyond the review's four commands, two rows pin the same defect in the publish and sleep rows.
  `bash -c 'gh pr view'` was a false deny at HEAD, because the gh word class took the closing quote.
- F4: the top-level go alternative keeps the base word end, and a single quote is a word end only in a single-quoted script.
- F5: the script reaches and the script branches of `redirectWritePattern` read a heredoc operator's line.
  The raw go row matches an expansion of a lym variable inside a double-quoted string of a single-quoted script.
  A double-quoted string inside a double-quoted script stays unread, and the `rawGoCommandPattern` and `commandPositionPattern` comments and the claudeengine package doc say so.
- F6 and the rows F1, F4 and F5 name: added to `TestRows_PatternsDenyAndAllow`.
- F3 (`internal/shuttleengine/claudeengine/doc.go`): the raw go, refuse-raw, shell-edit, computed-rm and headless claude pass lists no longer name `bash -c`, and the `sed -i` after `do` or `then` clause is dropped.
- `internal/shuttleengine/claudeengine/testdata/settings-golden.json` is regenerated.
  A temporary test wrote it, and the test is deleted.
- Verified: `lym gate test --tier 1 ./internal/shellguard` passed, and tier 0 passed over claudeengine, codexengine, configcli and orchcli.

- Gate repair: the comments this round added in `rows.go` and the claudeengine `doc.go` that wrapped at a fixed column or ran to 120 characters are rewritten one sentence per line, each under 120 characters.
  No code changed.
- Second gate repair: the claudeengine `doc.go` sentence on a double-quoted script's inner expansion broke mid-phrase.
  It is now two sentences, one per line.

Commits: `0f36837b` (F3), `e900bf0a` (F1, F4, F5, F6), `5599a424` (the F5 comment fix from the self-check), `fb72303d` (first gate repair), and the second gate-repair commit that follows them.
F1, F4, F5 and F6 share one commit, since the scoped patterns they change are one rewrite.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, the `commandPositionPattern` comment and the claudeengine doc claimed the script is read the same way with no exception for a double-quoted script's inner expansion. Fixed as proposed in `5599a424`.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

- F2: the parent ruling (decision notice #2155, "Shell keyword positions widen top-level rows too") accepts the keywords do, then, else, if, elif, while and until as command positions at the top level.
  The target already holds it, so the ruling asks for no change and none was made.
  The parent was not told, since nothing needed flagging.

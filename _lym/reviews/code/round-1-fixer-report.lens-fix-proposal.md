```yaml
lens: fix-proposal
outcome: reported
```

no findings

Nothing breaks F1–F5 as proposed. These are the cases I built:

- **F3 regex `(?:[0-9]*[<>]&[0-9-]*|&>|[^;&|()\n`])*`:**
  - Alternation priority takes the redirect form at each `2>&1`, `>&2` and `&>`.
  - A lone `&`, `&&` or `;` still ends the segment, so `lym gate test x 2>&1 & ls other-weft` and `lym gate test >&2 && ls other-weft` still flag `other-weft`.
  - I found no input where the gate segment now swallows a real command.
- **F1 rows:**
  - `echo lym gate other-weft` returns true only if blanking is confined to command position.
  - The `;`, `|`, `||` and newline rows each fail if the segment runs past that separator.
- **F2 git-read-failure row:** `headErr` reaches `acceptAuditWayForward` and not an earlier call.
  - The other `HeadSHA` calls sit in `recoverSpawn` (`recoverbatch.go:263`), which runs after the refusal, and in `committedSince` (`recoverbatch.go:455`), which needs a recorded `RecoveryStartSHA`.
  - The row's fixture sets no `RecoveryStartSHA`.
  - So the row fails if the function stops returning the error.
- **F5 nil-guard row:** the entry is the mutating `lym fabric add x`, so without the guard `readOnly(cmd)` would panic on the nil classifier. The row therefore pins the guard.
- **F4 wording and test row:**
  - "any command other than a `lym gate` invocation" matches `MatchPath`'s behaviour. Gate blanking only removes the bare-word match, and the separator-prefixed path rule still runs.
  - The new Verify row uses slug `alpha`, so it exercises the gate exemption and not the card-id finding.

Further tool calls (2 of 6):
1. `grep` for `HeadSHA|acceptAuditWayForward|UncommittedPaths` in `recoverbatch.go`: my context did not show the other `HeadSHA` call sites.
2. `awk` and `grep` over `recoverbatch.go` lines 185–266 and 440–458 and its function headers: to place those call sites relative to the refusal at line 541.

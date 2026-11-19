```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F3, F4, F5, F6).

Proposal list: F1 (script-bounded argument scans), F3 (doc.go lists), F4 (top-level `go` word end), F5 (script expansion and heredoc branches), F6 (extra rows). I found one finding, LOW.

**LOW — the `commandPositionPattern` doc comment in `internal/shellguard/rows.go` still claims more than the code reads**
- F5's fix said that if a top-level refusal was left out of the script, the `commandPositionPattern` comment must say so.
- The fixer added only the matching note to the `rawGoCommandPattern` comment: "A double-quoted string inside a double-quoted script is not read."
- The `commandPositionPattern` comment is unchanged. It still says the script "is read the same way" and "The guards thus refuse inside the script what they refuse outside it."
- That is false for a lym variable expanded inside a double-quoted string of a double-quoted script, e.g. `sh -c "echo \"$LYM_GATE_SLOT\""`.
- The top level denies that spelling and the double-quoted script does not.
- The claudeengine `doc.go` sentence "The script of a `sh -c` or `bash -c` in command position is read like the command itself, one level deep" carries the same overclaim.

Cases I built that did not break the proposals:
- **F1, double-quoted script:** `bash -c "claude --version" && echo 'run -p'` stays allowed. The argument word excludes `\`, so the scan stops at the script's closing `\"`.
- **F1, `gh pr view` ending a script:** `bash -c 'gh pr view'` is now allowed, because the single-quote scope's `ghWordChars` excludes `'`. The fixer added a row for it.
- **F4:** a top-level `go test 'all'` was allowed at base and stays allowed. Nothing newly denied at the top level.
- **F5, expansion alternative:** it cannot reach past the script's closing `'`, because the preceding units never cross an unescaped `'`.
- **F5, `<<<` here-string:** it behaves as at the top level, so the heredoc branch adds no new false deny.
- **Pattern length:**
  - The longest settings line in the golden grew from 24,583 to 38,264 characters, well under the 128 KB single-argument limit.
  - The golden never renders the publish row, which is the largest one and is now tripled by scope.
  - Its real size is unmeasured, but the integration test passes it to `grep` through `sh -c` without error.

Calls made:
- `git show 08ebbcf6:…/settings-golden.json | awk` plus the same `awk` on the current golden: my context lacked the golden's line lengths, which the pattern-length case needed.
- `grep -rn publishCommandPattern|DenyPublish`: my context lacked where the publish row is installed, which the pattern-length case needed. It showed the row is a launch-spec switch and absent from the golden.

```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the F1–F4 change set.

Findings left unchecked: none.

no findings

- F1: no remaining mention of `claudeengine`, the golden or its test command in `internal/shellguard`'s docs. The only hits are `lym gate test` strings in steer text and test rows.
- F2: the one code literal of the golden path is the `settingsGoldenPath` declaration. The other hits under `docs/research` are recorded measurement data.
- F3: no "separator, a quote or the payload's end" wording or `searchEnd` remains. The word unit and the run end both use `unquotedWordBreakers`. Other copies of the same character set (`rows.go:247`, `:478`, `ghWordClass`) are not part of the exception scan F3 concerns.
- F4: the branch diff touches only `internal/shellguard` and `internal/shuttleengine/claudeengine`, and the blast radius line names both.

Tool calls:
- grep of `internal/shellguard/*.go` for other packages' documentation (F1): my context had no search of the changed package docs.
- grep for golden path literals (F2): the earlier search predates the change.
- grep for end-set wording and character-set sites (F3): the earlier search did not cover these strings.
- branch diff stat and the Merge danger lines (F4): my context lacked the branch's changed-file list.

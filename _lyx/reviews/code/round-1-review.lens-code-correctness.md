```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, which means running each changed function on concrete inputs.

**Per-function list**

- `pattern.EntryRuleText`, bare line `- \`PATTERN-a\` — When: x`. The head matches, the head and ` — ` are trimmed, and the result is `When: x`. Correct.
- `pattern.EntryRuleText`, line with `(test)` and a link. The link tail is stripped first, then ` (test)`. Correct, because the link is always last.
- `pattern.EntryRuleText`, line with a link only. The link is stripped and no marker is trimmed. Correct.
- `pattern.EntryRuleText`, line with `(test)` and no link, or no text at all. Both give the right result. For an empty line, `TrimPrefix` leaves it empty and the length is 0, so it is not over the cap.
- `pattern.EntryRuleText`, name-less line `- loose text (test)`. There is no head match, so the bullet is trimmed and the marker removed, giving `loose text`. Correct. Check's own entry-head handling is unchanged.
- `pattern.EntryRuleText`, name with no ` — ` separator, e.g. ``- `PATTERN-a` text``. Nothing is trimmed after the head, so the text keeps a leading space (` text`). It is one character too long at most, and the format check flags the missing separator separately. Not a defect.
- `Check`'s long-line branch is correct. A line whose whole length is over 360 but whose rule text fits no longer fires. A text of 281 fires, and the message reads "entry rule text is %d characters, over the %d cap".
- `patternRuleText` in discussionparser, on the same rows as `EntryRuleText`. `parsePatternBullet` has already stripped the ` — added after Discussion (…)` suffix and trimmed the text, so the text ends in the link or `(test)` as it would on an overview line. Link-then-marker order matches `EntryRuleText`, and the shared-row test pins this. Correct.
- `patternRuleText`, text ending in ` (test) — added after…` with the suffix stripped. The suffix is removed before the cap check, so the marker is cut as intended. Correct.
- `patternOverCapFindings`, `removes` skip. This is unchanged. Boundary: 280 runes passes and 281 fails, with a multibyte `é` counted as one rune.
- `patternOverCapFindings`, the `wayForward` branch for appended bullets. This is unchanged and still reachable. The test `AppendedOverCap` pins it.
- `checkPatternEntryLineCap` in planparser, entry at 280 behind a long name, marker and link. The rule text is 280, so it passes. At 281 it fails and the length reported is 281. Correct.
- `checkPatternEntryLineCap` with a prescribed line from `CardPatternEntries` that has a name but no separator. This behaves like the `EntryRuleText` case above, which is harmless.
- `TestValidate_PatternEntryLineCap` helper: `Sscanf` on `"rule text at "`. It matches the new detail string, so the length is parsed correctly.
- `TestValidate_PatternChangeOverCap`: `textFor` now yields `cap+extra` runes. `RemovesNeverMeasured` uses a long id and `removes` is skipped, so it stays valid.
- `append_test`'s `PatternTextOverCap` uses `cap+1`. The old value of exactly 360 filler runes was over the cap under the old whole-line rule. The new value is right for the new rule.
- `parity_test.go`'s `overCap` uses `MaxEntryTextChars+1`, so it is over the cap as the test needs.

**Findings:** none.

**Tool calls**
1. Read `internal/discussionparser/patternchanges.go` lines 95–150. The manifest lacked `parsePatternBullet`, and I needed it to confirm that the appended suffix is stripped and the text trimmed before the cap check.

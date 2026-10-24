```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the `--from` batch change in `decision.go` and `append.go`.

**Per function and branch: case run, verdict**

- `decisionEntries`, one-entry path: empty title, and a partial PATTERN form (`--pattern-text` alone). Both are refused with a way forward. Correct.
- `decisionEntries`, `--from` beside a per-entry flag: refused before the file is read. Correct.
- `decisionEntries`, unreadable file, empty file (`io.EOF`), `{}`, `["a"]`, `[] []`: all refused. Correct. JSON `null` decodes to a nil slice and gets "holds no entry". Correct.
- `decisionEntries`, unknown key (`{"titel": ...}`, or `"patern_text"` misspelt): **wrong**. The decoder has no `DisallowUnknownFields`, so the key is silently dropped. See finding 1.
- `decisionEntries`, empty field, partial PATTERN form, repeated title (trimmed), repeated PATTERN entry ID (trimmed): refused with the entry number. Correct.
- `decisionEntries`, trailing garbage: `[…] ]` or `[…] }` is accepted, because `Decoder.More()` returns false when the next byte is `]` or `}`. Wrong but marginal. See finding 2.
- `AppendDecisions`, empty `ds`: error, nothing written. Correct.
- `AppendDecisions`, a later entry with an empty field, or a missing `## Decisions` heading: the error comes before the write and the record is unchanged. Correct. `ErrNoDecisionsHeading` is still wrapped with the path.
- `AppendDecisions`, a finding or an error from `Validate`: the prior bytes are restored. Correct.
- `insertDecision`, two entries with PATTERN bullets: the second call re-derives offsets from the already-extended content, so order is kept and the second bullet follows the first. Correct, and the new test pins it.
- `AppendDecision`: a faithful one-entry case. Correct.
- `decisionNoticeLine`, one entry: identical to the old text. Correct.
- `decisionNoticeLine`, several entries, inline or short: one line with headings. Correct.
- `decisionNoticeInline`: `MerriamStrandRole` is `"webster"` (`internal/websterengine/roles.go:39`), matched through `MatchesRole`, which is exact or a `-N` suffix. Merriam's strand gets the texts, and the open risk in the record is a strand-naming risk only. Correct.
- `notifyLiveRuns`: one post per live receiver, and a failed post does not stop the next. Correct.
- `beginAmendment`, a failure on title k: `restoreAmendment` puts the whole file back to its prior bytes (or removes it if it did not exist). Correct.
- `beginAmendment`, repeated `AppendAmendment` calls: each read-appends-writes, so the titles come out in order. Correct.
- `rollBackAppend`, on findings or an error: the amendment is restored, and the decision record only after an error, since the append restores it on findings. The status-commit case is handled. Correct.
- `afterFailedCommit`, `committedLacksEntry` over the batch: it reports the first missing heading, and a missing HEAD record counts as lacking the first entry. Correct.
- `decisionVerb`, the darn post-commit check: verifies every entry's heading is in the committed record. Correct.
- `decisionVerb`, envelope: `headings` always; `pattern_changes` when any; `heading` and `pattern_change` only for one entry. Correct. `Bullet()` is trimmed, as before.
- `decisionVerb`, a whitespace-only `pattern_entry` (for example `" "` with action `adds`): it passes the `!= ""` pairing check and becomes a `PatternChange` with an empty ID, after which the record's discussion check decides. This matches the one-entry path's untrimmed check, so it is not new. Marginal. See finding 3.

**Findings**

1. **HIGH · design** · `internal/loomcli/decision.go:148-155` (`decisionEntries`)
   - A `--from` file with an unknown or misspelt key is not refused. The entry is silently decoded with that field empty, or the PATTERN change is dropped.
   - The decision record requires unknown keys to be refused (Constraints bullet on `PATTERN-tolerant-records`, and acceptance criterion 4).
   - Evidence: `json.NewDecoder(bytes.NewReader(data))` is never given `DisallowUnknownFields()`. `grep -n "DisallowUnknown\|unknown"` over `decision.go` and `decision_test.go` returns nothing, so no test reaches this refusal either.
   - Example: `{"title":"a","decision":"d","rationale":"r","pattern_enty":"PATTERN-x","pattern_action":"adds"}` is accepted as a plain entry. The `pattern_action` alone is then caught by the pairing check, but `{"…","patern_text":"t"}` and `{"tittle":…}` with other keys present give a misleading "empty title" or lose the text silently.

2. **LOW · design** · `decision.go:150`
   - `decoder.More()` is used to detect data after the array, but it returns false when the next byte is `]` or `}`. A file such as `[…] ]` or `[…] }` is accepted.
   - A stricter check is a second `Decode` that must return `io.EOF`.

3. **NIT · design** · `decision.go:168,588`
   - The PATTERN-form pairing check and the `if entry.PatternEntry != ""` branch use the untrimmed value, while the repeated-ID and `PatternChange.ID` handling trim it. A whitespace-only `pattern_entry` therefore builds a `PatternChange` with an empty ID. It is inherited from the one-entry path.

**Tool calls**
1. One `Bash` call (`grep` of `MerriamStrandRole`, `MatchesRole` and `AppendAmendment`). My context lacked the Merriam role value, the role-matching rule, and whether repeated `AppendAmendment` calls accumulate titles in order.

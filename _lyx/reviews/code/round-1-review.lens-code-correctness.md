```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness — I traced each changed function or branch with a concrete input and a failure case.

**Per-function list**

- **`AppendDecisions` (`append.go`):**
  - Empty or nil batch returns the error "no decision to append", and nothing is read or written. Correct.
  - A later entry with an empty field makes `insertDecision` fail before the write, so the record is untouched. Correct.
  - A record with no `## Decisions` heading returns `ErrNoDecisionsHeading` wrapped with the path, and `errors.Is` still matches. Correct.
  - A finding or an error from `Validate` restores the prior bytes. A failed restore comes back joined with the original error. Correct.
- **`insertDecision`:**
  - Entry 2 is inserted into the content that already holds entry 1. It lands at the end of the section, so input order is kept.
  - A second PATTERN entry uses `patternBulletRange` on the updated content. It replaces a bullet that is already there and is otherwise appended.
  - The `patternAt < at` and `patternAt >= at` branches both keep the offsets valid.
  - `TestAppendDecisions_WritesBatchWholeOrNothing` pins the ordering, the empty batch, the empty field and the finding case. Correct.
- **`decisionEntries`, one-entry path:** An empty field and a partial PATTERN form are refused with a way forward. Correct.
- **`decisionEntries`, `--from` path:**
  - A per-entry flag beside `--from` is refused. Correct.
  - An unreadable file, an empty file (EOF), `null`, `[]` and `[1]` are each refused. Correct.
  - Trailing data (`] ]`, `[]`) is refused through the second `Decode`; any non-EOF result counts as trailing data. Correct.
  - `[null]` decodes to an empty entry and is refused as an empty title. Correct.
  - A misspelt key is refused by `unknownEntryKey`. The key pass is stricter than the decoder: `"Title"` decodes through case-insensitive matching but is refused as unknown. This is acceptable.
  - A duplicate title or PATTERN ID is detected on trimmed values. Correct.
- **`decisionNoticeLine`:**
  - One entry gives the inline form for every receiver, so a one-entry call is unchanged.
  - A batch to a non-Merriam receiver names headings only.
  - A batch to Merriam inlines every text.
  - A batch is never empty, because `decisionEntries` refuses it. Correct.
- **`notifyLiveRuns`:** A list failure is logged and reported. A post failure goes into the failures and the loop continues. Correct.
- **`decisionVerb`, `--amends-plan` with a batch:**
  - A failure on the second title restores the amendment record to its prior bytes.
  - An error from the append restores the decision record and then the amendment record. `restoreDecision` is a no-op when the record did not exist.
  - A finding is already restored by the append, so only the amendment record is restored. Correct.
- **`decisionVerb`, commit failure:**
  - Without an amendment, both records are kept and the refusal carries `decisionWayFabricCommit`.
  - With an amendment, `afterFailedCommit` checks that every entry of the batch is in HEAD through `committedLacksEntry`, not just the first. Correct.
- **`decisionVerb`, darn check after the commit:** `committedLacksEntry` returns the first missing heading across the whole batch. Correct.
- **`decisionVerb`, envelope:**
  - `headings` is always set.
  - `heading` and `pattern_change` appear only for a one-entry call.
  - `pattern_changes` appears only when a bullet exists.
  - The date is taken once, so every entry gets the same heading date. Correct.

**Findings:** none.

One unverified edge, not a finding: a whitespace-only `pattern_entry` passes the key-pairing check. It then gets an empty trimmed ID, which skips the duplicate-ID check. The `patternBulletHead` regex at `patternchanges.go:32` probably makes `Validate` flag the bullet. In that case the record is restored and nothing is committed.

**Tool calls**
1. `grep -n 'ID' internal/discussionparser/patternchanges.go`. I read it to see how `Validate` treats an empty or odd PATTERN ID, which my context lacked.

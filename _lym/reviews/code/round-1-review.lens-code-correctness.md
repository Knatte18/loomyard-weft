```yaml
lens: code-correctness
outcome: reported
```

Task: review `internal/loomcli`, `discussionparser`, `fabricengine`, `gitrepo` and `loomrecipe` for correctness, one concrete case per changed function or branch.

**Per-function list**

| Function or branch | Case ran | Result |
|---|---|---|
| `decisionCallerRefusal` | Arbiter label from a non-arbiter strand; sampler strand with `--by parent`; sampler read error with a parent caller; parent unresolved; unnamed shell | Correct, same outcomes as the inlined code it replaces. |
| `holdDecisionLock` | Lock directory missing; a second holder | Correct: `MkdirAll` runs first and `AcquireWriteLock` blocks. |
| `decisionVerb` and `decisionStoreVerb` lock | Lock failure; early refusal after the lock is taken | Correct: `defer release` covers every return. |
| `decisionReplaceVerb` guards | Empty flags, unreadable file, wrong `--by`, Discussion-Write running, approval row not passed, span, record absent, record uncommitted, base mismatch (upper-case hash), working copy differs | Correct. The lowercase and trim on the base hash is right. |
| `replaceFileRefusal` | Malformed `## PATTERN changes` bullet | Correct: `CheckRecordSections` includes `ParsePatternChanges`, so the check `Validate` makes is covered. |
| `CheckCarryOversKept` | Dropped, changed and added block; no `## Open risks`; malformed markers in either record | Correct. |
| `CheckAddedEntriesSubset` | Entry dropped; rewritten; inserted; fenced heading; the same entry twice | Wrong for the duplicate case, see finding 1. |
| `landReplace` | Commit ok; commit staged nothing and the empty commit fails; commit errors with the bytes at HEAD and the trailer; with the bytes and no trailer; without the bytes; HEAD unreadable; `headCarries` error | Correct on each branch. The restore runs only when HEAD lacks the bytes, and every refusal names a way forward. One residual, see finding 2. |
| `headCarriesReplace` | Newest trailer commit is HEAD with an equal value, or is not HEAD | Correct. |
| `raddleFilingRefusal` | No Raddle-File row; perch at the row before filing; non-perch before filing; store marker done; marker missing but entry not a task; marker missing but board holds the parts; proposal absent or unparseable; no part slugs | Correct, matching D3. The `isTask && !held` gate sends an absent proposal to "not filed", as intended. |
| `boardCopyDue` and `copyRecordToBoard` | Past approval; approval row, not past; no approval row with and without a stored record; beam project record; `boardRecordStored` error | Correct. The error is reported as `record_store_error` and never fails the verb. |
| `decisionStoreVerb` | Filed raddle run; no approval row and no stored record; approval row and not past | Correct. |
| `recordHistory`, `newestReplaceOf`, `cutAtReplace` | Another record's newer trailer; sha matches no version; bytes landed before the trailer commit; sweep after the replace; empty trailer commit over an earlier sweep; same bytes at an older position | Correct. The `Position > trailer.Position` skip and the newest-first scan give the D2 scenarios. |
| `parseRecordReplaced` | Path or value with a space | Splits wrongly on a path with a space. The record path is `_lym/...` and slug-built, so this is unreachable. |
| `AnchoredFileFirstParentHistory` | Unborn HEAD; deletion commit; commits sharing a committer second | Correct. `gitrepo.Log` with `FirstParent` and `PathPrefix` compares each commit to its first parent in first-parent order, as D2 needs (`historyreads.go:177-239`). |
| `TrailerCommits`, `FindTrailerCommit` | Unborn weft; no match; newest and oldest | Correct. Positions are `len-1-i`, and the oldest is the last entry. |
| `CommitEmptyWithTrailers` | `SkipGit`; invalid trailer; path unchanged with another path staged | Correct. The pathspec helper is the same one `StageAndCommit` uses, so no new slash mismatch. |
| `CommitPathAllowEmpty` | Unchanged path; another path staged; a path matching nothing | Correct. Git refuses and the error is returned wrapped. |
| `InImplementerSpan` | Each tapestry row; a retired row; an empty row; darn; unknown recipe | Correct. Only tapestry has both a `PlanWrite` row and Webster rows. |
| `writeCarryOverLocked` | Write error; commit error | Correct: the lock is held across both and released by `defer`. |

**Findings**

1. LOW, `design`, `internal/discussionparser/replacecheck.go:240-262`.
   `CheckAddedEntriesSubset` tests membership in a set of entry texts, so a `--from` file holding the same existing added entry twice, or reordered, passes.
   D1 says the file "adds none", and a duplicate adds an entry.
   Each added entry is also compared after `addedDecisionSpanText` trims trailing whitespace, so a difference that is only trailing whitespace on an entry's last line passes.
   Both go against "byte for byte".
   Fix: count entries, or require the file's entries to be an ordered subsequence of the record's.

2. LOW, `design`, `internal/loomcli/decision.go:742-746` (`landReplace` restore path).
   After a failed commit with HEAD lacking the bytes, only the working copy is rewritten to `prior`.
   `CommitRecordsPaths` may have staged the file's bytes before the commit failed, so the index can keep them staged against restored working bytes.
   This is the same exposure `add` already has, and a later path-scoped commit takes the working copy, so it is not harmful today.
   I could not confirm from the code I read whether `StageAndCommit` unstages on failure.

3. NIT, `design`, `internal/fabricengine/trailercommit.go:509-526` and `internal/loomcli/projectrecord.go:708-723`.
   Every `recordHistory` call now runs `TrailerCommits`, a full first-parent log of the records branch with messages, and a cut history then runs `AnchoredFileFirstParentHistory`, which makes two more full walks.
   On a long weft this repeats per gate round.
   It is not incorrect.

No other correctness defect found.

**Tool calls**
1. Grep of `Validate` and `CheckRecordSections` in `internal/discussionparser`: to see whether the replace check includes the PATTERN-bullet check; the manifest held only `doc.go`.
2. Grep of `PathPrefix` in `internal/gitrepo`, plus `sed` of `commitweftpaths.go:47-79`: to check the pathspec the empty trailer commit and `CommitRecordsPaths` share; the manifest did not show them.
3. Failed grep of `internal/gitrepo/log*.go`; no such file.
4. Read of `internal/gitrepo/historyreads.go:120-270`: to confirm `Log` honours first-parent plus `PathPrefix`, which `AnchoredFileFirstParentHistory` relies on and the manifest did not cover.

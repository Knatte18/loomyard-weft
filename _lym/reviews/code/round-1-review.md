---
verdict: APPROVED
---

Range reviewed: `git diff d026a9a8c37ec705c5c7185cba2c23f79ad854f2..HEAD`, one commit.
Judged against the decision record's `## Goal`, the board entry for issue #33.

Both dirty paths the entry names are covered:

- The deleted `_lym/loom/friction/loom-halt.md` falls under the new `loomengine.LoomFrictionDirRel()` exclusion.
  A pathspec exclusion hides a tracked deletion as well as untracked files, and the test commits a note and then deletes it to pin that.
- The untracked `_lym/shed/<slug>/go-settle.jsonl` sits under `shedrun.RunsRootRel()` (`_lym/shed`), which `runStateExemptRels` already exempted before this diff.
  The diff leaves it exempt, and the test's `runState` list still writes `SettleRecords`.

The archive glob `_lym/loom/friction-*` matches each timestamped archive sibling and everything beneath it, because git's default pathspec `*` crosses `/`.
It is joined under `AnchorRel` by `ScopedPathspec` and slash-normalised by `exemptPathspec`, so a subpath anchor and Windows separators both work.
`gitexec.Run` sets no literal-pathspec mode on the `Clean` path.

Scope holds: only the friction directory and its archives are added, not the whole `_lym/loom` directory.
The `runStateExemptRels` comment states that the directory also holds notes agents write.
The commit message and the change description match the diff, and no `Pattern-Departure:` trailer or declared PATTERN change applies.

## Rejected

- Lens code-tests and lens code-affected-set, LOW: no direct row for `LoomFrictionArchivesRelPattern` in `internal/loomengine/friction_test.go`. `TestCheckResolved_LymOwnStateIsExempt` pins the glob at both anchors through the public surface, and a per-helper test for a one-line function is what the rubric's test economy rule flags.
- Lens code-affected-set, LOW: the glob matches a bare `friction-` name that `IsLoomFrictionArchive` rejects. lym writes only timestamped archive directories there, so no reachable input differs, and the effect of a stray entry is only that it is exempt.

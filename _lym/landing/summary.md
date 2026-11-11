# lym loom decision replace folds a run's decision record under a lock

`lym loom decision replace` lets a run's parent change or withdraw an earlier decision by writing a whole, checked decision record.
It folds `Added after Discussion` entries into the main decisions.
Until now the only post-Discussion writer was `decision add`, which appends; nothing could fold the record or drop a stale call.
The dropped-entry check also held every added entry forever.

## What changes

**The `replace` verb** (`internal/loomcli/decision.go`).
`lym loom decision replace [<slug>] --by parent|operator|arbiter --from <file> --title <t> --base-sha256 <hex>`:

- It takes `add`'s caller guards, now shared as `decisionCallerRefusal`, and refuses on every run `add` refuses.
- It is also refused before a Discussion approval the recipe has.
- It is refused between the plan's approval and the recipe's last Webster row, where each fresh Webster prompt renders only added entries.
  `loomrecipe.InImplementerSpan` reads that span from the recipe's rows.
  Both refusals name `lym loom decision add`.
- The record committed at HEAD must hash to `--base-sha256`, and the working copy must equal it.
  The refusal names no hash; its way forward is to re-read, rebuild and re-run.
- The file must pass the section check and `CheckDecisionIDs`.
  It must keep every carry-over block byte for byte (`discussionparser.CheckCarryOversKept`).
  Each of its added entries must equal one of the record's (`discussionparser.CheckAddedEntriesSubset`).
  So it may drop entries and appended PATTERN bullets, but never insert or rewrite one.
- The commit goes through `CommitAnchoredPathsWithTrailers` with a `Lym-Record-Replaced: <by> <path> <sha256>` trailer.
  When nothing is staged because HEAD already holds the bytes, `fabricengine.CommitEmptyWithTrailers` lands an empty commit carrying the trailer.
  It commits the record's path alone and leaves any other staged path staged.
- When the commit errors, `landReplace` reads HEAD:
  - the written bytes under this call's trailer mean success, with `commit_error` reported;
  - the bytes without the trailer get the empty trailer commit;
  - only a HEAD without the bytes restores the prior record, and the refusal names re-running `replace`.
- It refreshes the board copy as `add` does.
  Each live agent gets a notice opening `Parent decision record replaced with lym loom decision replace` that names the title and asks for a whole re-read.

**One per-run decision lock.**
`add`, `replace`, `store` and the Bouncer's carry-over writer (`writeCarryOverLocked`) all take `holdDecisionLock`.
It is a blocking file lock whose path `lock.TrackedFileLockPath` forms beside the record.
The verbs hold it from their first precondition through the commit and the board copy, so a base check and its write are atomic against the other writers.

**The history check stops at the newest replace** (`recordHistory` in `projectrecord.go`).
It lists the records branch's first-parent commits that carry the trailer (`fabricengine.TrailerCommits`).
It takes the newest trailer whose path names this record and reads the file's first-parent history (`fabricengine.AnchoredFileFirstParentHistory`).
It keeps the versions from the cut onward.
The cut is the newest version hashing to the trailer's sha256 at or before the trailer commit's first-parent position.
Ordering is by first-parent position, not committer time.
A later commit of the same bytes is never the cut, so an entry added after a replace and then dropped is still reported.
With no matching trailer, or a sum no earlier version has, the history is read whole as before.

**Every stored record is refreshed** (D3).
`add` and `replace` copy the record to the board past a Discussion approval, as before.
On a recipe with no approval row, such as raddle or a raddle-filed darn part, they now copy when the entry carries a stored record.
A beam's project record is never copied.
`store` is admitted on the same runs.
`raddleFilingRefusal` replaces `perchFilingRefusal`.
Once a raddle run's filing is done it refuses `add`, `replace` and `store` in any mode at any row, naming the filed parts and `lym board upsert --record-file` for the family's project.
The filing counts as done when the store's marker is missing but the entry is no longer a task, or the board already holds the proposal's parts.
That refusal names `lym loom resume` first.

**Docs.**
- The discussion spec's "Parts no writer authors" says `replace` may remove but never edit an added entry, and that the check reads back to the newest replace.
- The parent directive quotes the replace notice's opening.
- The parent directive tells a parent when to `add`, when to `replace`, to fold a Sketch with `--record-file`, and to re-`add` folded calls before a goto to a Webster row.
- `docs/overview.md` names both verbs and the flag.
- `PATTERN-run-records` names `replace` as a project record writer.

## Design decisions

- **Webster span.**
  The span check lives in `internal/loomrecipe` (`InImplementerSpan`), outside the record's package list.
  It reads the recipe's own rows: the plan approval is the Bouncer row Plan-Write hands to, and Webster rows are `Implementer` rows.
  That follows the existing `PastDiscussionApproval` instead of matching row names in `loomcli`.
- **Lock file.**
  The decision lock is `<record lock place>.decision.lock` under the never-tracked lock directory, so it never collides with the record's other locks.
- **Filed check.**
  The board judgement uses the proposal's part slugs alone, not the project slug, as the record says.
  An absent board entry counts as still a task, so a removed entry never refuses on its own.
- **Restore on error.**
  When HEAD cannot be read after a failed commit, `replace` refuses without restoring the record, since it cannot tell whether the bytes landed.
- **Parent directive wording.**
  The stencil renders into a spawned role, so the four parent instructions sit under "When you are the parent of a run yourself".

## Tests

- `TestDecisionReplaceVerb_WritesCommitsAndNotifies` covers each admitted caller.
- `TestDecisionReplaceVerb_Refusals` covers every refusal, asserting the record, HEAD and the board are unchanged and the base refusal carries no hash.
- `TestDecisionReplaceVerb_CommitOutcomes` covers the commit outcomes.
- `TestDecisionLock_SerializesRecordWriters` uses a real file lock to show an `add` and a carry-over write both wait for a `replace`.
- `TestDecisionCalls_RefreshEveryStoredRecord` and `TestDecisionCalls_RefusedOnAFiledRaddleRun` cover D3.
- `TestRecordHistoryCut` pins each D2 scenario.
- `TestRecordHistory_CutsAtTheNewestReplace` drives the history read over real weft commits.
- Fabric and gitrepo integration tests pin the new seams, including an empty trailer commit while another path stays staged.
- `TestInImplementerSpan` pins the recipe span.

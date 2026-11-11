# board upsert --record-file replaces a stored record; promote warns

A Sketch keeps its approved decision record on the board as `decision-<slug>.md`.
Until now nothing but a Discussion approval could rewrite that file,
so an operator who edited the Sketch's body had no way to bring the record along.
This change adds that way, and a warning at promote time when the two have drifted apart.

## `lym board upsert --record-file <file>`

The flag replaces an existing entry's stored record in the same board write as the payload's other fields.
`boardengine.Board.UpsertTaskWithRecord` does the write under the board lock:
the payload is validated by the store's upsert,
then the record file is written, and only then is `board.json` saved.
This is the order `StoreDecisionRecord` uses.
When a check after the mutation refuses the write, the former record file is written back,
the same way `FileRaddle` restores a record.

Before anything is written, the CLI runs two checks from `internal/discussionparser` on the file:
`CheckRecordSections` (new) and `CheckDecisionIDs`.
`CheckRecordSections` is the record-only half of `Validate`: the required H2 sections and the `## PATTERN changes` bullets, with no support log, since the board keeps none.
`Validate` now calls it, so the two cannot drift.

The engine refuses the following, each with a way forward and nothing written:

- a run's task session, by the board's existing strand-name classification;
- a new entry, an entry with no stored record, and a done entry;
- an entry a run holds, which gets `lym loom decision replace <slug>` on the run as its way forward;
- an entry whose slug has a task worktree in the hub, because Record-Write never overwrites the record a worktree holds.
  Its way forward is `decision replace` in that worktree, or `lym batten cancel` first;
- an entry whose weft branch is live, and an upsert whose origin probe fails.

A family project whose raddle filing the raddle store records is exempt from the worktree and weft refusals, since its run reads no record again.
The guard reads no batten liveness.
It does not see a run started after its check, or one on another machine.

## One liveness test for Add and the upsert

The live-weft refusal uses `fabricengine.Topology.RecordsBranchLive` (new).
The name says "records branch" because the fabric vocabulary keeps the weft token inside fabric's owner set.
It reports whether a slug's weft branch is live by the test `Topology.Add` adopts by:
a local branch, or a branch on origin that no `archive/<slug>/*` tag covers.
Both go through one unexported probe, `weftLiveness`, which applies `Add`'s own `SkipGit`/`SkipPush` skip,
so the upsert refuses exactly when a raddle start would adopt the weft.
A local branch decides with no origin read.
The CLI passes the skip from the fabric skip environment, as every fabric verb does, and itself runs no git.
The probe runs before the board lock is taken, since it may wait on the network.
The entry refusals are judged again under the lock.

## `record_body_sha256` and the promote warning

Every write of an entry's stored record stamps the entry's new `record_body_sha256` field.
The stamp is the sha256 of the body that write leaves.
The writers are `StoreDecisionRecord`, the record upsert above, and Raddle-File's filing write, both for filed parts and for a family's project.
The field lives on the entry in `board.json`, so a board sync's replay carries it, and an upsert payload naming it is refused as an unknown field.
`StoreDecisionRecord`'s no-op shortcut now also requires the stamp to match the body.
So re-storing the same record after a body edit refreshes the stamp.

`lym board promote` of a Sketch whose body, as loaded before the write, no longer matches its stamp adds a `warning` to the envelope.
The warning names `lym board upsert --record-file`.
It never refuses.
An entry with no stamp, stored by an older binary, gets no warning.
To carry the warning, `Board.PromoteTo` now returns a `PromoteResult` holding the task, the siblings and the warning.

## Design decisions

- The record upsert's refusals live in `boardengine`, beside the store they judge, not in `boardcli`.
  The CLI owns only the file read, the format checks and the probe wiring.
  Judging the entry under the board lock is what makes the guard hold against a concurrent write.
- The run-held refusal and the worktree refusal name `lym loom decision replace`, as the decision record asks.
  That verb lands with the board entry that depends on this one, `decision-record-replace-verb`.
- The warning fires on any body change after the store, one that leaves the design alone included.
  A body hash cannot tell the two apart.
- `lym board upsert --help` keeps its long page under the CLI form's size cap.
  So the page describes `--record-file` only in the flag's own help line, and the package doc holds the full rules.

## Testing

- `discussionparser`: `TestCheckRecordSections` pins the bytes-only check's findings, each with an empty path.
- `fabricengine`: `TestRecordsBranchLive` covers a local branch, an origin-only branch with and without a covering archive tag, no branch, the `SkipPush` skip, and an unreadable origin.
- `boardengine`: the record upsert's refusals, each asserting its way forward and an unchanged record and entry.
  The tests also cover the successful replace with its stamp, the family-project exemption, and a refused payload leaving the former record in place, before and after the record file was written.
  They pin the stamp on `StoreDecisionRecord` and `FileRaddle`, and the promote warning's three cases.
- `boardcli`: `TestCLI` steps drive the flag end to end.
  They cover the replace, the file's format refusals, the probe-failure refusal without the fabric skip, the run-session refusal, and promote's `warning` key.

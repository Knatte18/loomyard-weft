# Card 1 — Status-strand removal helper

**Edit:**
- `internal/loomcli/bootstrap.go`
- `internal/loomcli/bootstrap_test.go`

**Create:**
- `plan:internal/loomcli#removeStatusStrands` -> `func removeStatusStrands(status func() (reedengine.StatusResult, error), remove func(guid string, recursive bool) (reedengine.Removed, error))`

**Uses:**
- `internal/loomcli#statusStrandDisplayName`

**Intent:**
Add the llm arm's status-strand removal as a standalone function in `bootstrap.go`, beside `statusStrandDisplayName`, `findStatusStrand` and `resolveStatusStrandAction`, so it is testable offline before any verb calls it.

`removeStatusStrands` takes reed's own `Status` and `RemoveStrand` method shapes as function values, the same way `reedDriverPaneProbe` holds them, so production passes `c.reed.Status` and `c.reed.RemoveStrand` directly and a test substitutes recording fakes.
It reads the strand table once through `status`, then calls `remove(guid, false)` for every strand whose `Name` equals `statusStrandDisplayName` — every one, not the first, because an older build may have left a duplicate.
The removal is always non-recursive: reed refuses a non-recursive removal of a strand with children and removes nothing, so a status strand with anything parented beneath it stays up instead of cascading through strands this task never meant to touch.
It returns nothing and never fails its caller: a failed `status` read logs one `logger.Warn` and removes nothing, and a failed `remove` logs a `logger.Warn` naming the guid and carries on to the next same-named strand.
The doc comment states the warn-and-continue stance (a leftover band costs the operator screen rows, not the run — the same stance `ensureStatusStrand` takes on a failed `ReplaceStrand`) and why the cascade argument is `false`.

Bundle table-driven tests in `bootstrap_test.go` covering:
- no strand carries the name: `remove` is never called;
- one status strand among other strands (including the driver strand and a `loom-status-extra` prefix-sharing name): exactly that guid is removed, and no other;
- two strands carry the name: both guids are removed, in table order;
- every `remove` call passes `recursive == false`;
- `status` fails: `remove` is never called and the function returns normally;
- the first of two `remove` calls fails: the second same-named strand is still removed.

**ImpactSummary:** Adds one unexported function and its tests in `internal/loomcli`; no caller yet, so no existing behaviour changes.

**Commit:** 1: loomcli: add removeStatusStrands for the llm arm's status-strand cleanup

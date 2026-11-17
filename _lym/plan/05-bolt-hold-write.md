# Card 5 — bolt-hold-write

**Create:**
- `internal/boardengine#HoldWrite`
- `internal/boardengine#WriteHold`
- `internal/boardengine#TestWriteHold`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#pushOwnerCommit`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture`

**Intent:** Add the one hold write, the only writer of `board.json`'s `holds` list, which the upgrade's first read calls (D5, D1's synchronous Bolt write).

`WriteHold(boardDir, hold)` writes an `adopt` or `adopt-fix` hold in one pull-first Bolt commit and pushes it synchronously through `pushOwnerCommit`, as `RegisterMachine` does.
After the pull it judges the holds that stand:
a hold of another machine stands, so it writes nothing and returns that hold in `HoldWrite.Standing` with `Wrote` false;
a hold of the same machine with the same kind and subjects stands, so it writes nothing and returns `Wrote` false with no `Standing`;
otherwise it appends the hold and returns `Wrote` true.
That makes a re-run of a cut first read idempotent, and a hold another machine pushed meanwhile wins.
A hold whose kind is neither `HoldKindAdopt` nor `HoldKindAdoptFix`, or an `adopt-fix` hold with no `FixTask`, is refused as a wiring error naming the field.
An unreachable origin returns `ErrOwnerOriginUnreachable` wrapped; `BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH` behave as in `SetOwners`.
Clearing a hold is not here: a green check clears it in the adopt part (D6).

`TestWriteHold`, tier1, runs on `newOwnerFixture`'s hub: a written hold is on origin when it returns; the same hold again writes nothing; a hold another clone pushed first is returned as standing and nothing is written.
`doc.go` gains the setter.

**ImpactSummary:** New setter only; nothing calls it until the upgrade's first read in card 22.

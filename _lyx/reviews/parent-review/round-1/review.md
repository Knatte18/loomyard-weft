---
verdict: APPROVED
---

## Scope against the board entry

The board entry `claimed-scope-edit` asks for five things, and the decision record covers each:

- Darn-Write writes a short decision record in loom's format in the same session, before code: D1, D2, D3, with D4's `discussion` gate enforcing it.
- `lyx loom decision add --by parent` works on a darn run once the record exists: D5, with D6 telling the writer how to follow a recorded correction and D11 telling the orch to record one.
- The Darn review reads the record and treats the amended scope as authoritative over the board body: D7, with D8 carrying the perch's open findings into the record.
- First check where the Darn review reads its scope today: answered in the support log (Interview item 3, Q1), the live board body through `lyx board get`.
- PR-Align in darn, after checking whether it was left out on purpose and what the alignment agent reads that darn lacks: D9 and D10, with the check answered in Interview item 4 and A1.

The record's Out list matches the entry's direction.
Editing a claimed entry's body was an option the operator's direction passed over, and splitting darn into Discussion-Write, Plan-Write and Webster rows is the option the operator dropped.

D7 narrows the entry's wording "holds it authoritative over the board body" to the record's `Added after Discussion` entries, and treats the writer's own sections as a claim.
This is not a scope deviation.
The entry's own "Other options considered" states the requirement as treating the amended scope as authoritative over the original, and the parent's appended entries are that amended scope.
The record states the narrowing and its reason in D7.

D2 (the format spec), D4 (the gate and Go commit), D8 (carry-over) and D11 (orch stencil) go beyond the entry's literal asks.
Each is a necessary consequence of the asked work, and none is work another board entry owns:
`PATTERN-producer-pointer-rule` needs D2, the record must exist before a correction arrives (D4), PR-Align's `pr-gate` judge reads carried-over open risks (D8), and the orch's guidance otherwise keeps sending corrections only by message (D11).

## Other board entries

- `orch-split` (note, not running) restructures the orch role stencil that D11 edits.
  It decides nothing about `decision add` or PR-Gate guidance, so there is no duplication or contradiction.
  The record names the overlap under Open risks.
- `lighter-test-rules` (note) is an audit of stencil-wording test pins.
  This run moves pinned text, and the record names that cost under Open risks.
  There is no conflict.
- `discussion-premise-cites` (note) would extend `validate-discussion`, which darn's record would then pass through too.
  It decides its own reach, so there is no conflict today.
- `glyph-use-audit` (note) is where the `webster-pack-last` incident behind this entry came from.
  This run does not touch its subject.

## Overlap with runs in flight

`lyx loom peers` lists two peers and no parent commits.
`own_row` is 3.

- `twill-spawn-after` (darn, Darn-Burler-Review, `row` 0): an unknown order.
  Its diff against the fork base changes `contracts/stencils/loom/loom-template-discussion.md` in the header's marker comment and the write-scope list, where it drops `twill_write_scope`.
  It also changes `docs/overview.md`, `contracts/stencils/stencils.go`, `contracts/stencils/claims_test.go` and `contracts/stencils/markers_test.go`, which this run's D2 is likely to touch when it registers the new spec and adds a specs-directory marker.
  This run moves the stencil's Step 5 record format into `contracts/specs/loom-discussion-spec.md`.
  The overlap is in the text only: the same files, at different places or as adjacent registry and marker edits.
  `twill-spawn-after` reorders the fork procedure and drops the ready marker, and decides nothing about the record format, darn's record, `decision add` or PR-Align.
  It has no decision record to contradict.
  The record names this merge under Open risks.
- `tender` (loom, Webster, `row` 12): it is further along, so `tender` lands first and this run takes its changes at Merge-In.
  The only shared file is the bookkeeping file `cmd/lyx/testdata/test-budget.yaml`, together with `internal/battencli/wire_seams_test.go`, a shared seam test.
  `tender` changes pause, resume, revive and stale-binary restart, and none of its paths touch darn, alignment, the decision record or the stencils this run edits.
  There is no design conflict.

No parent commits are listed.

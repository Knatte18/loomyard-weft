# Card 3 — The arbiter refuses a Merge-In or Publish verify halt whose own record names a target

**Edit:**
- `internal/loomcli#arbiterStop`
- `internal/loomcli#arbiterEligibility`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterCall.checkEligible`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D2: a `blocked` stop at Merge-In or Publish whose verify-failure record names a target and belongs to the halt being judged is not eligible to the arbiter, since batten's first-verify-failure retry, and then the parent, owns it.

`arbiterStop.PublishHaltNamesTarget` is renamed `VerifyHaltNamesTarget`, documented as: a blocked stop at Merge-In or Publish carries a verify-failure record that names a target and belongs to the halt.
`arbiterEligibility` refuses on it with "a Merge-In or Publish verify halt whose failure record names a target belongs to batten's retry and then to the parent", ending with the hand-back way forward as every refusal there does; its doc comment names both rows.

`arbiterDeps.PublishFailureTarget` is replaced by two seams, the ones the Go driver's deps already carry:
`VerifyRecord func() (battenshed.ChildPublishFailure, bool, error)`, the verify-failure record and whether one exists, and `Head func() (string, error)`, the pair's HEAD.
`loomCLI.arbiterDepsFor` fills them exactly as `internal/loomcli/drivewiring.go` fills the driver's `VerifyRecord` and `Head`: `verifytree.ReadPublishFailure` mapped to `battenshed.ChildPublishFailure`, and `fabricengine.Open(location)`'s `HeadSHA`.

`arbiterCall.checkEligible`, for a `blocked` status whose current producer is `loomshed.NameMergeIn` or `loomshed.NamePublish`, reads the record; when one exists and names a target it reads HEAD and sets `VerifyHaltNamesTarget` to `battenshed.RecordMatchesHalt(record, head, c.status, []string{loomshed.NameMergeIn, loomshed.NamePublish})`, the match battenshed's retry makes before acting: the record's row is the halting entry's and the current producer, its head is HEAD, and it was written after the history entry before the halt and no later than the halt.
A read error of either seam refuses as the record read error does today ("the Publish failure record" becomes "the verify failure record"), a raw I/O failure on the section's grouped transient row.

Bound: the refusal removes the arbiter only from a Merge-In or Publish verify halt whose own failure record names a target.
Every other blocked stop at Merge-In and Publish stays eligible, a stale record from an earlier verify included, which narrows today's Publish refusal: a blocked Publish stop of another cause with a stale record naming a target becomes eligible.
With no batten, or with batten's one retry spent or capped, such a Merge-In halt now parks for the parent as the Publish halt already does; the LLM driver acts on it through its existing arbiter-refusal path, and its stencil is unchanged.

`arbiterFixture` replaces `publishTarget` with `verifyRecord *battenshed.ChildPublishFailure` (nil: no record) and `head string`; `arbiterFixture.deps` wires `VerifyRecord` and `Head` from them.
`TestArbiterVerb_SpawnPath`'s rows set `f.status.History` with `At` times (RFC 3339, whole seconds) ending in a `shedengine.Stuck` entry of the halting row, `f.head`, and a record whose `Head` and `WrittenAt` place it inside or outside the halt:
- "Publish halt naming a target refused" keeps its name, now with a matching Publish record; it wants the new refusal text "verify halt whose failure record names a target".
- "Merge-In verify halt naming a target refused": the same at `loomshed.NameMergeIn`.
- "Merge-In stop of another cause with an earlier verify's record stays eligible": the record's `WrittenAt` is at or before the entry before the halt, so it spawns.
- "Publish stop of another cause with an earlier verify's record is eligible": the same at Publish.
- "blocked stop at another row with a record naming a target stays eligible": a `Plan-Write` halt with a record whose row is `Plan-Write`.
The existing "Publish halt naming no target" row stays eligible with no record.
Eligibility reads no producer list, so `newArbiterFixture`'s rows are unchanged.

`contracts/specs/refusal-spec.md`'s row "arbiter: stop not eligible, Publish halt naming a target" is renamed "arbiter: stop not eligible, verify halt naming a target", its trigger naming the Merge-In or Publish row and the belongs-to-the-halt match; no second row is added.
`internal/loomcli/doc.go`'s `# Arbiter` eligibility sentence says "a halted `blocked` stop other than a Merge-In or Publish verify halt whose own failure record names a target".
The old refusal text "Publish halt that names a target" is asserted only in `TestArbiterVerb_SpawnPath`.

**ImpactSummary:** The arbiter verb's eligibility input, its two deps seams and their one production wiring in `loomcli`, one table test with its fixture, one spec row and one doc sentence; the Go driver's deps and batten are untouched.

**Commit:** 3: Refuse the arbiter at a Merge-In or Publish verify halt owned by batten's retry

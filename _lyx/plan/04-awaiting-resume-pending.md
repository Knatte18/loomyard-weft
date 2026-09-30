# Card 4 — awaiting-resume-pending

**Edit:**
- `internal/battenshed#innerRunProducer.callAwaiting`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#approvalActedFile`
- `internal/battenshed#approvalActedFileSuffix`
- `internal/battenshed/innerrun_awaiting_test.go`

**Uses:**
- `plan:internal/battenshed#approvalActedContent`
- `plan:internal/battenshed#parseApprovalActed`
- `internal/battenshed#approvalIdentity`
- `internal/battenshed#newInnerRunDeps`
- `internal/battenshed#newRunShed`

**Intent:**
Tell "resumed, the child's driver has not re-stepped yet" apart from "the child halted at a hand-off again" in `callAwaiting`, so the poll right after an approved resume stops telling the operator to re-approve (D7, D8, D9).

On a successful resume, write `approvalActedContent(identity, len(status.History))` to the marker instead of the bare identity, where `status` is the child status this Call read and passed in.
On the check, parse an existing marker with `parseApprovalActed` and compare its identity with the current approval's identity alone.
When the identities match:

- `hasLen` true and `len(status.History)` equal to the recorded length: the resume was delivered and the child's driver has not re-stepped yet.
  Return `exemptWait` with a reason saying the approval at `<approved_at>` was acted on, the resume was delivered, and the child's driver has not re-stepped yet;
  and that if this persists, a live driver is inspected in its pane, while an ended one is replaced by running `lyx loom start` in the task worktree.
  The reason never calls `lyx loom start` a re-delivery and never names `lyx loom approve`.
- any other length, or an old-layout marker (`hasLen` false): the existing re-approve hint, text unchanged.

A non-matching identity resumes as today, whatever length the marker recorded.
The `ErrChildNotParked` arm still writes no marker.
The recorded length is a clock-free discriminator because a re-await appends at least the child's own Awaiting entry, and card 1's fold never folds onto an Awaiting entry.

Doc comments: `approvalActedFileSuffix` and `approvalActedFile` describe the marker as the approval identity line followed by the child's history length at that resume, with a one-line marker read as the old layout;
`Call`'s disposition list splits the "approval already acted on" bullet into the not-yet-re-stepped wait and the awaiting-again re-approve hint, both budget-exempt with no spawn;
`callAwaiting`'s own doc comment names the three awaiting outcomes.

In `innerrun_awaiting_test.go`:

- `TestInnerRun_AwaitingResumesOncePerApproval` gives the awaiting child a non-empty history, asserts the marker holds exactly `approvalIdentity(approval)` followed by that length and a newline, and changes its second-Call expectation to the not-yet-re-stepped reason: it contains the approval time and `lyx loom start` and names the pane, does not contain `lyx loom approve`, and spawns nothing; its third Call, with a new approval and the child's history length still equal to the recorded one, still resumes (D9), even though that length would otherwise mean the child has not re-stepped yet.
- a new test scripts the child's history one entry longer on the second Call and expects the existing re-approve hint with no second spawn;
- a new test pre-writes an old-layout marker (the identity line alone) for an awaiting child and expects the re-approve hint with no spawn;
- a new test drives many awaiting polls with no approval through `newRunShed`, and asserts from the persisted status that the history holds the seeded counted entries plus one `Run-Shed` budget-exempt Stuck whose `Repeats` is the poll count minus one, and that no step blocks.
- the existing `ErrChildNotParked` and spawn-error tests stay as they are and keep passing.

**ImpactSummary:** Confined to battenshed's awaiting arm and its marker; the not-parked and new-approval paths are unchanged, and no other package reads the marker.

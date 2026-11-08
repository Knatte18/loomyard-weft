# The beam's orch writes its PR description; beam describe records it

A beam's change description is now written by the beam's orch, not assembled by lym.
`lym beam describe <slug>` used to rewrite `summary.md` whole from the entry's title and brief, the beam's commits and every `Added after Discussion` entry of the project record.
That decision list was a history of the way there, partly duplicated and partly superseded, and no reader of the pull request needed it.
The orch could change the text only through the entry's title and brief.

## What changes

- `lym beam describe <slug>` now records the description the orch wrote at `summary.md` in the beam's `_lym/landing` directory.
  It validates the file, stamps the landing record's `DescribedAt` and commits both in one commit, as before.
  When the landing record names a pull request, it then runs the `lym beam pr update` sequence, so the pull request carries the text.
  A failed update is refused saying the description is recorded, with the update's own way forward.
- `lym beam describe <slug> --draft` writes a starting point over `summary.md` and records nothing.
  The draft holds the entry's title and brief, then the beam's commits with each member pull request's number, title and merge time.
  It never draws on the project record.
- A missing description is refused naming `--draft`; a validation finding is refused naming the file and the verb to run again.
- The orch role's beam file (`contracts/orchrole/role-beams.md`) tells the orch what to write:
  the end result, never the way there; every member pull request; never the decisions made along the way.
  The orch and the operator may change any part of the text at any time and run the verb again.
- The dobby notice for landings into a beam, the pull request follow-up and `lym beam ready`'s refusal of a failing description now name editing the description and `lym beam describe`, not `lym board upsert` or a separate `pr update`.

In `internal/landingshed`, `WriteBeamDescription` and `BeamDescribeDeps` give way to `DraftBeamDescription` with `BeamDraftDeps`, and `RecordBeamDescription`.
In `internal/beamcli`, the `Describe` seam splits into `DraftDescription` and `RecordDescription`.

## Design decisions

- **The draft replaces `summary.md` whole.**
  A beam worktree can inherit a `summary.md` from its parent, so a draft that refused an existing file would block a fresh beam.
  The verb's help says to edit the draft before recording, so the orch drafts once and edits after.
- **Recording updates a recorded pull request in the same verb.**
  The design asks the verb to push the text to the pull request.
  It reuses `prUpdate` whole, so the owner check, the branch push and the draft-to-ready step stay in one place.
  `lym beam pr update` stays for a push with no text change.
- **No content check for a decision list.**
  The instructions in the role file and the verb's help keep the decision list out; the verb validates only the summary format.

## Testing

- `TestDraftBeamDescription` and `TestRecordBeamDescription` cover the draft's content, the replaced file, the missing and failing descriptions with their ways forward, and the failed seams.
- `TestDescribe` covers the draft, the record with and without a recorded pull request, a failed pull request update and the existing refusals.
- `TestHubLandDeps_DraftThenRecordStampsAndCommits` drafts and records over a real hub, checking the listed landing, the stamp and the one commit.

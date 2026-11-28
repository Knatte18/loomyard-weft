---
round: 1
focus:
  - "internal/webstercli/reset.go, the fabricSync failure branch: does every way the sync can fail after the records commit landed reach the records push, and does `committed` (internal/webstercli/sync.go, CommitWithTrailers) report that correctly, including SkipGit, SkipPush and standalone wiring?"
  - "internal/webstercli/reset.go, the same branch: what does the caller learn when the push there fails, a hand-over included? Check the envelope, the logging and the named way forward against the verb's help text, the package doc and the normal path."
  - "internal/webstercli/reset.go, the pushResetRecords extraction: does the success path keep its behavior for skip conditions, hand-over, warning log and `push_error`?"
  - "internal/webstercli/reset_test.go, row SyncFailingAfterTheCommitLeavesAnOwedRecord: read the whole test body. Does the new row assert the refusal, `push_error`, the owed record and the transition commit, does it fail without the fix, and do the other rows keep their checks? Compare each assertion with what the change description says the row requires."
  - "_lym/landing/summary.md, `## Merge danger`: check the door and blast radius against the rubric's Merge danger bullet, given that the diff adds a records push on a path that had none."
  - "pattern/ and PATTERN.md statements about the reset's synchronous records push and `PushOwed`, and the reset.go header, help text and doc.go: do they agree with the code as the diff leaves it?"
  - "Commit messages of the review range: does the `Root-Cause:` trailer name the defect the diff removes?"
---
Seeded from the board entry under `## Goal` and a first reading of the range.
The record holds no `Added after Discussion` entry, so the entry's Fix paragraph is the settled scope:
run the records push before refusing when the commit landed, and pin it with a test row.

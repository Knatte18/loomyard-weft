---
round: 1
focus:
  - "internal/websterengine/audit.go readOnlySegment (card 44): does any allowlisted shape still fail open? Settle whether filepath.Base(words[0]) admits a path-spelled program (./cat, /some/dir/grep) as read-only, whether a readOnlyGit subcommand can still run a command or write through its own options (git grep -O/--open-files-in-pager, a -c config value) beyond the --output scan, and whether commandWrappers, timeout and xargs option skipping can misread a value word so that a mutating wrapped command reads as read-only."
  - "internal/websterengine/audit.go mutatingCommand's empty-command fail-closed branch and commit 8c4b50f2b: can the fork or parent audit (audit.go:178, audit.go:256) record a fabric-reference finding with an empty or unparseable Command in practice, and if so, does the change to a correctness class turn a benign finding into a failed batch or a demoted run exit?"
  - "Card 44 tests in auditseverity_test.go, recordbatch_test.go and runlevel_test.go: is every row the card names present with its stated class, does every rewritten FABRICREF fixture still reach the policy path its test asserts, and does the new unlisted-writer test assert a terminal failed batch, an archived report and a digest reason naming the command?"
  - "internal/shedadapters/webster.go ErrPendingAuditFindings branch (card 46): read the composed Reason against runlevel.go:1033's message. Does err.Error() plus ', then re-step the <row> row' leave one trailing way-forward clause, and does the refusal-spec 'pending audit findings' row quote that clause as the code emits it? Through loomshed's websterProducer wrapper, does the loom recipe's Webster row (no on_stuck) end blocked rather than failed, and do the two new webster_test.go tests prove that?"
  - "internal/webstercli/acceptaudit_test.go TestAcceptAuditCmd_FabricSyncFailureNamesWayForward (card 45): does it clear the skip-git switch so the sync actually runs, assert a non-zero exit, the error text, the sentinel and 'lyx fabric commit', and re-read state.json to find the finding accepted and none pending? Does the refusal-spec 'fabric sync failed' row match card 45's trigger wording?"
  - "Comments new or changed by cards 44-46 (audit.go's mutatingCommand, readOnlySegment and ClassifyViolation docs; the readOnlyTools comment; websterengine/doc.go; shedadapters/doc.go's WebsterProducer bullet; webster.go's new inline comment; the new test comments): check them against CONSTRAINTS.md's Comment Line-Break Convention, which treats the surrounding fixed-column wrapping as legacy rather than a convention to match."
  - "Whole diff at HEAD: do go build, go vet (including -tags smoke), go test ./... and go test -tags integration ./... pass, in particular for websterengine, webstercli and shedadapters, which cards 44-46 and commit 8c4b50f2b touch after the two earlier approved passes?"
---

Two earlier Webster-Review passes (archived under `_lyx/reviews/webster-20260930T191503Z` and `_lyx/reviews/webster-20261001T143735Z`) approved the diff through card 43 and its fix commits.
Since then the branch added cards 44–46, which fix PR #321's round-2 findings A, B and C, plus follow-up commit `8c4b50f2b`.
None of that has been reviewed, so round 1 starts there.
Every focus entry is still a question to settle against the whole range, `git diff $(git merge-base main HEAD)..HEAD`, with `main` taken from `_lyx/shed/refusal-audit/status.json`.

Cards 44–46 have only Edit and Prosa groups, so they carry no Rename or Delete mechanical check.
Card 44 still deletes `mutatingSegment`, `editsInPlace` and `mutatingTools`.
A repo-wide grep at seed time found no remaining reference to any of them.

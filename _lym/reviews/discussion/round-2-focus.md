---
round: 2
exclude_lenses: []
focus:
  - "Index row source (decision-record.md D5, D6; internal/landingshed/finalize.go recordLanding and HeadSHA; internal/loomcli/landingdeps.go RecordLanding; shedrun.RecordLandedCommit): does every row field, the landing commit and its committer time included, come from a source that a resumed Finalize reads with the same value after the parent moved on, on the already-landed and GitHub-merge paths too?"
  - "Index row ordering (decision-record.md D5; internal/landingshed/finalize.go landUnderLock, pushParent, pushRecords, markTaskDone, dropBeamRecord): is the row written where the pull request URL is still readable and where Finalize's own commit and push carry it, and what holds on a crash between those steps?"
  - "Fixer and fix review copies (decision-record.md D4; internal/landingshed/fixerrun.go, fixerattempts.go, fixreview.go): is every turn's fixer report and every fix review round's report kept, under names that cannot collide across turns, row re-entry, a removed attempt record, an escalate or a failed start?"
  - "Kept directory under concurrent commits (decision-record.md D1, D3; internal/fsx/fsx.go AtomicWriteBytes; internal/loomcli/wiring.go status commit staging; internal/fabricengine/remove.go commitPendingRecords): can a temp or half-written file be staged or reported as dirty, and is any claim about git's behaviour there pinned?"
  - "Records across a machine move (decision-record.md D2, D3, D4; internal/burlerengine/parentreview.go ParentReviewFilesOf and RecordParentAsk): for each kept record whose live source or numbering is under .lym, can a new owner machine or a re-run overwrite or duplicate what origin already holds, pr-findings re-runs included (internal/loomcli/prfindings.go)?"
  - "Listing archive tags (decision-record.md D5, D6; internal/battencli/cancel.go; internal/fabricengine/archive.go; internal/reportengine/runs.go): how is each kind of tag shown, including a cancelled run, an orphan pair and a diverged teardown's second tag, and does anything present a run that never landed as landed?"
  - "Attack surface of --extract (decision-record.md D6): does the Decision state what bounds the writes against symlink and submodule entries in the tag tree, a target directory that is a symlink or not a directory, a --tag outside the given slug's archive tags, and a failure part-way through?"
  - "Statements about existing behaviour (decision-record.md D5, D7, Constraints, PATTERN changes): do the claims about the landing commit's message, how the pr-findings commit is pushed (internal/loomcli/landingdeps.go pullRequestRecordCommitter), whether any code deletes archive tags (internal/fabricengine/cleanupremotewarp.go) and which PATTERN entry carries each new rule match the code and PATTERN.md?"
---

All lenses stay in the fan: each one raised a finding at MEDIUM or worse this round.

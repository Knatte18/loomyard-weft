---
round: 2
focus:
  - "Comment Line-Break Convention (CONSTRAINTS.md: one sentence per line, a further break at an independent-clause boundary, new and changed comments only, no line-width rule): settle whether each of these new or changed Go comments still joins two independent clauses with a semicolon on one line — internal/fabricengine/mergestage.go:131 ('Untracked files are never staged; MergeUntrackedFiles lists them…'), internal/fabricengine/mutation.go:83 (KindMergeTrackedStaged doc, 'mid-merge; Target is the warp checkout path. It is recorded…'), internal/websterengine/cardgate.go:2 ('Go derives the command and the fork runs it; Go never runs it…'), internal/websterengine/verifygatereport.go:64 and internal/websterengine/doc.go:401 ('The hint claims only that a card touched a failing package; Merriam judges it…'), internal/websterengine/runlevel.go:724 ('A done whose verify gate did not pass ends stuck; Master's own stuck…'), internal/websterengine/runlevel_test.go:113 ('…on this helper; a test that needs one calls appendIntegrationVerify…'). Then sweep the rest of `git diff $(git merge-base main HEAD)..HEAD` for any other wholly new comment line of the same shape, leaving edits inside legacy fixed-column paragraphs alone where a fix would rewrap untouched lines."
---

All six round-1 findings are fixed, and the build and test suites pass.
The one remaining question is whether the comment-convention sweep is complete.

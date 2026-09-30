---
round: 2
focus:
  - "Comment Line-Break Convention (CONSTRAINTS.md) in test files: do the new or changed comments in internal/loomshed/planwrite_test.go (seedPriorPlanStencils doc, the inline comment in MissingStencilWithPriorFilesIsAnErrorAndMovesNothing), internal/shedadapters/singlellm_test.go (the PrepareFreshSpawnErrorNeitherArchivesNorSpawns and PrepareFreshSpawnAmendment docs), internal/shedrecipe/entries_planwrite_test.go (TestPlanWriteEntry_CallAppendsPriorPlanBlock doc), internal/shuttleengine/claudeengine/prepare_test.go (TestPrepare_AppendSystemPromptMatchesDenyNotice doc), internal/loomrecipe/fixture_test.go (the edited seedBouncerStencils sentence) and internal/shuttleengine/claudeengine/command_test.go (the fork_mode_real_notice_pwsh inline comment) put one sentence per line, breaking only at independent-clause boundaries? Judge new or changed lines only."
  - "internal/loomshed/planwrite.go NewPlanDirRotator doc: does the sentence starting 'The closure's string result' break only at independent-clause boundaries, or does it still break after the colon before a fragment?"
  - "Any rewrap commits: are they comment-only, and do go build ./..., go vet -tags smoke ./... and go test -tags integration ./... still pass on the new HEAD?"
---

Round 1 fixed its eight findings, but the judge found that new comments in test files, and one rewrapped production doc sentence, still break the Comment Line-Break Convention.
Round 2 should confirm that those comments are rewrapped and that nothing else changed.

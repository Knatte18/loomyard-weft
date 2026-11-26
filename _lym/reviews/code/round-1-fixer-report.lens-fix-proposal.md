```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (fix-proposal lens).

1. **BLOCKING** (F1, the `--answer` refusal and the `resumeStep` text): a re-entered Master can no longer resume a waiting recovery honestly.
   - The unchanged resume section of `contracts/stencils/webster/webster-template-master.md` (the `waiting` bullet under "Read the trail by status") says: "The run was re-entered, so treat the step as taken: resume the batch with the verb named after `resume:`".
   - That verb now renders as `lym webster recover-batch N --answer "<the parent's answer>"`, from `waiting.go` `resumeStep`.
   - A fresh Master holds no parent message. It can pass the literal placeholder, which is non-blank, so it is accepted and recorded in `ParentAnswers` as the parent's answer. It can also invent an answer, or be refused.
   - This also hits the older `NEEDS_PARENT` step waits, which resumed with no input before this fix.
   - That bullet was not updated, for example to message the parent again before resuming.

2. **MEDIUM** (F1 and F2, the new section "A question only your parent can answer" in `webster-prefix-recovery.md`): it contradicts the parent directive rendered into the same prompt.
   - With a recorded parent, `contracts/stencils/parent/parent-directive-parent.md` says: "Ask it with `lym creel post --parent --expect-reply "<text>"`, then end your turn: your run waits for the answer". With the operator-ban line it adds that every operator question goes that way.
   - The new section says nothing answers a question you end your turn on, and to report `NEEDS_PARENT`.
   - A strand that follows the directive ends its turn with no report. It is classified `dead`/`asking` and stopped, and the parent's reply reaches nothing. That is F1's original failure.
   - The new section does not say that it overrides the directive.

3. **LOW** (F1): transient way forwards that can run over a waiting record still name the verb without `--answer`.
   - In `recoverSpawn`, the HEAD read, card-hash and start failures say "re-run `lym webster recover-batch %d`". The CLI's reed-boot refusal does the same.
   - On a waiting recovery, that exact command is now refused. The second refusal does name `--answer`.

4. **LOW** (F8 against F7): the new `AmendedCard` type doc in `internal/runplan/state.go` is wider than the behaviour.
   - It says "one card of a begun, not-done batch that a rebaseline accepted an edit to".
   - After the F7 limit, an accepted edit to a terminal `failed` or `stuck` batch records no entry, though that batch is begun and not done. The `AmendedCards` field doc states it correctly.

5. **LOW** (F7, the `else if bs.Status != DigestStatusDead { continue }` in `rebaseline.go`): a `failed` or `stuck` record that already holds an entry is no longer updated by a later `--card` edit.
   - Such a record comes from an in-flight amendment followed by a forced failure, or a stuck recovery that kept its entry.
   - A newly edited second card gets no entry, so `failureDigestBlock` tells the next recovery to re-read only the earlier card. A re-edit of the same card stays `Rendered: true`.
   - This matches the pre-change `continue` on terminal batches, and neither the rebaseline test rows nor the docs mention the held-entry case.

No case found against the F3, F4, F5, F6, F9 or F10 changes.

Tool calls (3 of 6):
- Read the `contracts/stencils/parent/` directive files and listed `internal/parentdirective`: the recovery prompt's parent directive text was not in context.
- Read the Master template's resume section and grepped `beginbatch.go` and `audit.go`: those template lines and the matcher's location were not in context.
- Read `internal/fabricengine/refscanner.go`: to check whether a quoted `--answer` text trips the parent audit. It blanks quoted command spellings, so no finding.

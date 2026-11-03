```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. Per guard, whether a real entry point reaches it.

**Card 1 (board-decision-record)**
- `StoreDecisionRecord` absent-slug refusal: reachable. The approve seam and the two verbs can pass a slug with no entry. See the card 7 finding for when this happens in a normal run.
- Done-entry refusal (`ErrRecordEntryDone`): reachable. A landed entry is done while its worktree and records still exist, so `decision add` and `decision store` can hit it.
- `noWrite` on same bytes: reachable.
- `DecisionRecord` read error when the record path is a directory: reachable (test-planted).
- `checkDiscussedPayload` on upsert, batch and merge: reachable through `lym board upsert/batch/merge`. On merge it runs on the store before the removals (store.go:1031 is the analogous `checkFrontPayload` site), so the merge-upsert-of-a-removed-slug row is reachable.
- Merge keeps the removed entry's `Discussed`: reachable.

**Card 2 (record-follows-entry)**
- Move on rename, and the move-failure branch (target is a directory): reachable.
- "Rename refused by a `ValidateSlug` seam leaves the file": the state is reachable through `checkNewSlugs` (board.go:144), which runs after the mutation and before the save. The card's fixture can't reach it, though (finding F2).
- Remove for a slug that `loaded` has and `after` lacks: reachable for remove, prune and merge.
- Removal-failure branch: reachable only by a planted failure. No test row reaches it, but it is a raw I/O error, which `refusal-spec.md` exempts as transient.
- Removal paths that skip `boardCriticalSection`: `ownerset.go:218` and `RetireLegacy` (board.go:465) save outside it but remove no entry, so they hold. Replay removal is covered by card 4 under D5.

**Card 3 (design-prefix-refusal)**
- `checkDesignPrefix` for `decision-`, `""`, `dec` and `decision-x-`: reachable. I found nothing in configengine that rejects or replaces an empty `design_prefix` before `LoadConfig`.
- The named way forward (`lym config board --set …`) works on a stack that already fails. `setChecked` only refuses a new load failure (configcli.go:278-293), so this holds. Card 3's claim that it "edits the file without `LoadConfig`" is wrong, but that wording is outside this lens.

**Card 4 (replay-carries-record)**
- Lost when a written record file's entry is not on origin: reachable (a commit that carries the file without its entry).
- Design-file rule for record files, removal included: reachable.
- A removal whose entry removal origin refuses or loses still deletes the file. That leaves a flagged entry with no file. It is reachable, but D5 chose it, so it is not a finding.

**Card 5 (board-get-record)**
- All four `--record` refusals (`--body` beside it, absent entry, undiscussed entry, flagged entry with no file) are reachable through `TestCLI`'s `GetRecord` step.

**Card 6 (discussion-approval-row)**
- No-approval-row (darn, pawl), not-past, empty-row, unknown-row and retired-row branches: reachable.
- Unknown-recipe error: reachable only through a direct call (see F3).
- Non-Bouncer `on_done`: reachable only through a fixture. No shipped recipe has it, and the rule test is meant for that.

**Card 7 (discussion-approve-seam)**
- `requireSeam(ApproveDiscussion)` and the unknown `approve_seam` value: reachable.
- `approveDiscussion` read error: reachable only in the test. At a real settle the working copy exists.
- `ErrRecordEntryDone` swallowed: reachable only when someone marks the entry done by hand mid-run. That is rare but reachable, and the real-test row covers it.
- Absent-entry and `OpenHub` failures that fail the settle: reachable in normal runs (finding F1).

**Card 8 (decision-add-copies)**
- `copyRecordToBoard` `pastApproval`-error branch: unreachable (F3).
- Way-forward fallback "or when it errs": unreachable for the same reason (F3).
- Committed record that HEAD does not hold, inside the copy: unreachable (F4).
- Done-entry branch: reachable.
- Other store-error branch: reachable.
- Failed-commit refusals naming the store verb: reachable through the no-amendment commit refusal and through `afterFailedCommit`.
- Landed-after-failed-commit path that copies: reachable.

**Card 9 (decision-store-verb)**
- The three caller-refusal shapes: reachable through the fake. The verb's real `deps.parent` is `decisionParent`, the same one `add` uses.
- No approval row, not past, status-read error: reachable.
- `approvalRow` error: reachable only for an unknown seed recipe.
- `pastApproval` error: unreachable (F3).
- No committed record: reachable after a `goto` past Discussion-Bouncer.
- Done entry and other store failures: reachable.

Findings:

- **F1. MEDIUM, decision.** Card 7, `approveDiscussion` plus `storeDecisionRecord`. The absent-entry refusal, and an `OpenHub` failure, are reached by every tapestry run whose worktree has no board entry, and each such run halts at Discussion-Bouncer on every resume. The run-status writer handles the same situation by writing nothing: `SetRunStatus` skips an absent or done entry (board.go:189-198), and a failed board status write only warns (wiring.go:216-223, the comment above `newCommitStatusSeam`). Card 7 makes only the done entry soft. D2 and D6 do prescribe that a failed store fails the settle, so the handler should weigh this against the answer key. Evidence: internal/boardengine/board.go:189-198, internal/loomcli/wiring.go:217-223, card 7 lines 71-73.

- **F2. LOW, consistency.** Card 2, `TestDecisionRecordFollowsEntry`, row "A rename refused by a `ValidateSlug` seam that refuses the target leaves the file at the old name". The card builds every row on `newRenameBoard`, whose `Config` sets no `ValidateSlug`, so `checkNewSlugs` refuses nothing on that fixture and the row can't reach the post-mutation refusal. The card should name a board config that carries a refusing `ValidateSlug`. Evidence: internal/boardengine/rename_test.go:24-27, internal/boardengine/board.go:144, card 2 lines 19 and 49.

- **F3. LOW, design.** Cards 8 and 9: the `pastApproval` error branches can't occur in the real verb.
  - `PastDiscussionApproval` can only fail when parsing an unknown or broken embedded recipe.
  - In `decision add`, `hasPlan` (`loomrecipe.HasEngine`) and `rowWritten` (`loomrecipe.RecordEngine`) read the same seed recipe first and refuse with "read the run's recipe". That makes these unreachable: card 8's `copyRecordToBoard` error text, the "or when it errs" fallback for `wayCommit`, and the `TestDecisionVerb_CopiesRecordToBoard` "pastApproval error" row.
  - In `decision store`, `approvalRow` (`DiscussionApprovalRow`) parses the same recipe before `pastApproval`, so card 9's "`pastApproval` error is refused" branch and its test row are unreachable too.
  - The upstream checks are `hasPlan`/`rowWritten` in `decisionVerb` and `approvalRow` in the store verb.
  - Evidence: internal/loomcli/decision.go:688-698 and 909-923; card 8 lines 44, 55 and 75; card 9 lines 48-52 and 75.

- **F4. LOW, design.** Card 8, `copyRecordToBoard`, "a committed record HEAD does not hold" branch and its test row. The copy runs only after `deps.commit()` succeeded, or after `afterFailedCommit` found the entries landed by checking `committedLacksEntry`. Either way HEAD holds the record, so the not-found branch can't occur in the verb. Only the read-error branch is reachable. Evidence: internal/loomcli/decision.go:782-800 and 585-591; card 8 lines 48 and 74.

Tool calls:
1. `.Save()` sites in boardengine, `newRenameBoard`, `resolveSeedRecipe`: to find removal paths outside `boardCriticalSection`, check the rename fixture's `ValidateSlug`, and see how the seed recipe resolves.
2. loomcli `runID` assignments and lymcwd `HubPath`: to see whether the approve seam's slug can differ from the board entry.
3. loomcli/cli.go 250-275: the run ID chosen from args.
4. Listing of internal/configengine: to find where empty layer values are handled.
5. grep of configengine for empty and null handling: to check whether `design_prefix: ""` is rejected before `LoadConfig`'s new check.

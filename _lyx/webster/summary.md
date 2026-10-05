# Bouncer review rounds escalate to the parent instead of halting for a human

All 16 batches recorded `done`, one card per batch, each committed.

## What was built

- **Ledger and evidence (01, 02):** ledger entries carry `class` and `severity`, validated against the vocabulary. `circlingEvidence` computes the gating keys open in a round and an earlier one from the ledgers' own round numbers.
- **Config (03, 04):** `loom.yaml` gains `review_circling_checkpoint` (default 3) and `review_max_bounces` (default 5). Review segments take their bounce budget from the latter, and the `Bounces` seam agrees with `Shed`.
- **Shed (05):** `parent_notice` rides on an `awaiting` halt through the status file and the step envelope, and `loom-status-spec.md` documents it.
- **Escalation (06, 07, 10):** two new Bouncer stencils (brief and parent notice), the `EscalationCause` vocabulary (`circling`, `budget`) and the per-round escalation record files. The Bouncer escalates `CIRCLING` and a spent budget, and settles from a recorded decision.
- **Circling verbs (08):** `lyx loom circling accept|continue` settle an escalation of either cause, record the `cause`, and read a legacy decision file as `circling`. The refusal row in `refusal-spec.md` follows.
- **Checkpoint judging (09):** the judge stencil's decision rule is rendered per checkpoint, and a Go guard reads an unearned `CIRCLING` as `CONTINUE`.
- **Exempt round (11):** the round after a budget `continue` spends no budget.
- **Decisions (12, 13, 14):** `discussionparser.AppendDecision`, the `lyx loom decision add` verb with its refusal rows, the Plan-Write stencil naming the verb, and `ly-drive` relaying `parent_notice` to `LYX_PARENT`.
- **Cleanup (15, 16):** the `recurring-finding` anomaly and its ledger plumbing are removed (`ledgeraccess.go` deleted). The loomcli smoke tier passes after strand matching moved to `agentname.Matches`.

## Deviations and notes

Files touched outside a card's declared file list, all to keep tests compiling or asserting current wording:

- 04: `internal/loomcli/progress_test.go`, plus a `reviewBudget()` helper in `arm.go` so the lightweight wiring does not see a zero budget.
- 08: `internal/shedadapters/bouncer_circling_test.go`.
- 09: `bouncer_judge_test.go`, `bouncer_verdict_test.go` and `bouncer_seed_test.go`. The fork also appended guard tests with a shell heredoc instead of `Edit`.
- 10: `internal/shedadapters/bouncer_verdict_test.go` and `internal/shedrecipe/entries_bouncer_test.go`. An unreadable escalation record returns a plain `Stuck`, a case the card does not cover.
- 15: `internal/loomcli/selfreport_github_test.go`.

Batch 13's first fork died on an API error (`EAI_AGAIN`) and `record-batch` classified it `no_report`. One re-fork with the same prompt found the earlier work complete, ran the gate and committed it.

Left open: `crucible/review-prompt-template.md` still lists "a recurring finding" among the anomalies that file a GitHub issue.

## Verify gate fixes

Attempt 1 failed on one test: `internal/reedengine.TestNaming_TaskWorktreeStrandNameTitleAndEnv` (`naming_integration_test.go:65`), with `show-options: exit status 1: invalid option: allow-set-title`.
The fixer fork found the installed tmux is 3.4, and the option likely needs 3.5 or newer; it did not check that against the changelog.
No card changed `internal/reedengine`, and `internal/reedengine/spawn.go:139` already treats a failed `set-option` as a warning.
The fixer made no commit: the only available changes were skipping the test or loosening its assertion, which the fix rules forbid.
The failure is environmental.
`ly:orch` answered that the hub already fixed the test on `main` in `7ff4cfc1`, which skips only the `allow-set-title` check when tmux reports "invalid option".
On the operator's instruction, that commit was cherry-picked into this worktree as `2b09f7e1` (`reedengine: naming test tolerates tmux without allow-set-title`), and the gate re-runs against it.

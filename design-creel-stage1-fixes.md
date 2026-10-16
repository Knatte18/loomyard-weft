# Fix the creel stage 1 audit defects and add its sandbox scenarios

`creel-stage1-fixes` · Task · bug, creel, shuttle, reed, claude · running · Plan-Burler-Review

Fix the defects the creel stage 1 audit found, and make its sandbox scenarios re-runnable.
The audit report is `docs/research/creel-audit-2026-10-15/report.md`; each issue below holds its steps and expected behaviour.

## Operator decisions (2026-10-15)

- The eight creel bugs from the audit are fixed together in this one task, at high priority.
- The audit's proposed sandbox scenarios C1-C13 are written into the sandbox suite in this task.
- `creel-notices` and `creel-agents` stay notes until this task has landed and creel is tested again.

## Defects

- #657: a strand added by hand with `lyx reed add --cmd claude` is never read idle, because Claude's greyed placeholder in the empty input box reads as a draft; `TestSmokeCreelClaudeStrand` (llm tier) fails for the same reason. Shuttle-launched sessions avoid it through `promptSuggestionEnabled: false`.
- #658: no delivery into a strand pane reed collapsed to 3 rows; the idle probe's `TooShort` is dropped in `internal/creelcli/delivery.go`, so the inject expires and no reason is shown anywhere.
- #659: after `interrupt` or `inject --interrupt`, Claude restores the interrupted prompt into its input box, creel reads the strand as busy, and the queued inject waits until the box is cleared by hand.
- #660: `lyx creel read > /dev/full` exits 0 and acknowledges, so the mail is marked read and lost.
- #661: a daemon that fails to start leaves verbs with only "no daemon answers"; its own error (for example a malformed journal line, with file and line) is lost.
- #662: a new inject is charged TTL for the time since the receiver's previous probe, before it was posted.
- #663: `read` does not show that a message was returned by a removed holder (`EndedUnreadBy` is never printed).
- #664: `log` prints `reminded` with `"id":0` and no ids; `inject --help` misstates who may inject; `--wait` verbs print an empty sender; `release` of an unknown receiver answers ok.

## Done when

- Each defect above is fixed as its issue's Expected section says, with a test where the repo's tiers can hold one.
- `lyx gate test --tags llm ./internal/creelcli` passes (`TestSmokeCreelClaudeStrand`), as do the tmux, unit and integration tiers.
- Scenarios C1-C13 from the report's "Proposed creel sandbox suite" are added to the sandbox suite in `tools/sandbox/`, beside the existing creel scenario S9.

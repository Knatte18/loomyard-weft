# Fixer report — discussion, round 1

Every finding's premise held against the code and the installed Claude Code binaries; all seven are fixed in `_lym/discussion/decision-record.md`, with the support log's ledger and rounds updated.

## Changes

- F1 (MEDIUM), fixed as proposed: D1 gains a rationale bullet that the continuation push is skipped only in simple mode (`CLAUDE_CODE_SIMPLE` or `--bare`, attachment also dropped), that the guard stands in 2.1.310, 2.1.311 and 2.1.312, and why no lym session reaches it; Open risks bullet 1 is deleted; Q10 is resolved (research).
  The scrub wording was narrowed in the self-check (see below).
- F2 (MEDIUM), fixed as proposed, first branch: D1 states the stop-hook summary row's 5-line clamp and judges it against the Goal (the operator sees the claim once, never twice; the model gets it whole), and the D1 real-session criterion checks a claim of more than 5 lines.
  The reviewer's mention of the full text in the transcript view was not carried over: the source read did not confirm it, and `lym creel log` prints no message body (`internal/creelcli/cli.go` `logCmd`).
- F3 (LOW), fixed as proposed: Open risks' version bullet records the cross-version guard and exit-2 evidence, Claude Code's self-update with no lym rerun, the failure a breaking update causes, and the rerun as an operator action.
- F4 (LOW), fixed as proposed in part: D4 states the wake's task-notification shape (summary tag holding `rewakeSummary`, body after it holding `<rewakeMessage> <stderr>`) and that the transcript row is built from the summary; the D3 real-session criterion requires exactly one operator copy and records whether the body is drawn.
  Whether the terminal draws the body is not settled from the minified source; see Deferred.
- F5 (LOW), fixed as proposed, then narrowed in the self-check: D3 releases the claim on every end of the ring's answer path before the daemon journals the acknowledgement; D5's `released` covers it; the D3 criterion checks it, and a plan-writer note names the seam its test needs.
- F6 (LOW), fixed as proposed: D3's second Bound names both loss windows, the lost confirmation after the journaled read included, each recorded by D5's `acknowledged` line; Constraints names both exceptions; D3's failed-acknowledgement sentence splits the journaled and unjournaled cases.
- F7 (NIT), fixed as proposed: D3 says the ring's own claim marks the ring answered at the acknowledgement, not at hand-out.

## Self-check

- fix-consistency, MEDIUM, D3's "a live session shows what the hook wrote" contradicts the unsettled body rendering: fixed as proposed, now "a live session's model receives what the hook wrote".
- fix-consistency, MEDIUM, the D3 criterion's "at most once" passes zero operator copies (merged with fix-way-forward MEDIUM and fix-proposal 3): fixed as proposed, the criterion requires exactly once and records whether the body is drawn, and the Open risk and Q11 name the next step for the not-drawn outcome: the ring's operator copy returns to the discussion.
- fix-consistency, LOW, the failed-acknowledgement sentence left the ring unanswered in the lost-confirmation window: fixed as proposed, the sentence splits the journaled and unjournaled cases.
- fix-way-forward, MEDIUM, Q11's settling step cannot show which outcome holds: merged above, fixed.
- fix-way-forward, LOW, the daemon-stopping release criterion has no deterministic test route: fixed, a Notes-for-the-plan-writer bullet names the seam that orders the outcome before the close or shutdown.
- fix-proposal 1, MEDIUM, the F5 release rule contradicts the F6 window: fixed as proposed, the release covers only ends before the daemon journals the acknowledgement, and D5 logs a lost confirmation after the journaled read as `acknowledged`.
- fix-proposal 2, MEDIUM, the scrub applies to the tmux server's environment at boot and the pane command sets `CLAUDE_CODE_` variables: fixed as proposed, confirmed at `internal/shuttleengine/claudeengine/command.go` (`forkSubagentEnvKey`); D1 now says the pane command never sets `CLAUDE_CODE_SIMPLE` and names the operator's own settings `env` key as the path outside lym.
- fix-proposal 3, MEDIUM: merged with the fix-consistency criterion finding, fixed.
- fix-proposal 4, LOW, the 5-line clamp runs over every Stop hook's lines: fixed as proposed, D1 and its criterion say the operator sees the claim's head within the row's 5 lines.
- fix-claims: no findings.

## Disputed

None.

## Deferred

- F4, in part: whether Claude Code 2.1.312's terminal draws an asyncRewake wake's body under its summary row is not settled by the minified source read this round (the row is built from the summary; the body's rendering site was not found).
  It is carried as Q11 and an Open risk, and the D3 real-session criterion, which requires exactly one operator copy, settles it; if the body is not drawn, the ring's operator copy is a design question for the discussion, not one this fixer can settle alone.

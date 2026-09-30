---
verdict: APPROVED
rationale: "All ten round-1 findings are legitimate under the rubric and each fix has landed in decision-record.md (D2 overview rule, D4 append-only, D6 fork and AskUserQuestion wording, D7 single-line constraint, plan.go comment in Scope and criteria, redundant risks removed); spot checks of settings.go, shell Quote, burlerengine and claude --help confirm the premises, and nothing new blocks."
---

# Discussion-Review, round 1 — judge verdict

## Review findings, ratified

The round-1 report raised ten findings (three MEDIUM, six LOW, one NIT).
None falls under the rubric's `Do not flag` list: none asks for the optional notes section, a rejected alternative in `decision-record.md`, or call-site enumeration.
Four are the relocation and exclusion kind the rubric puts on equal footing with gap-filling:

- F5, F8 and F9 remove Open risks and Auto-mode bullets that were rationale or duplicates.
- F7 moves the stale `plan.go` file comment out of the optional notes into Scope and Acceptance criteria.
  It passes the completeness-before-leanness test, since the notes section is non-binding.

F9 also passes that test: the `00-overview.md` requirement it found inside a to-be-deleted risk bullet was extracted into D2 before the bullet went.

The report has no `## Focus departures` section.
It covers all six round-1 focus entries: F1, F2, F3 and F4 answer the first three, F6 the fourth, F7 the fifth, and F5, F8 and F9 the sixth.

## Premises spot-checked

- `internal/shuttleengine/claudeengine/settings.go`: the AskUserQuestion steer reaches the session only as a `permissionDecisionReason`, the fork-mode Agent deny lets `subagent_type` `fork` through, and the webster guard is installed for every fork-mode run.
- `internal/burlerengine/engine.go:176` sets `ForkSubagents: p.ClusterFan != ""`, so the notice reaches burler cluster forks as well as the webster Master's forks (F3, F4).
- Both `internal/shell` `Quote` implementations escape a single quote inside single quotes (F2).
- `claude --help` documents `--system-prompt-snapshot` as recording the appended prompt until the conversation is compacted, and as having no effect where recording is not enabled (F5).
- `shuttleengine.AttachGated` collects candidates from `OutputFiles` only, which D4's attach-unchanged reasoning relies on.

## Fixes verified in the artifacts

- F1: D6's AskUserQuestion bullet carries the steer itself, and the acceptance criterion pins the final-message channel.
- F2: D7 requires a single line free of line breaks and `'"\`, with its rationale grounded in `shell.Shell` quoting, `tmux send-keys` and pwsh argument passing; the criterion covers line breaks.
- F3: D6 requires every sentence to hold inside forks, the fork-run sentence forbids further subagents, and a criterion pins it.
- F4: the webster-guard exclusion now rests on webster's fork stencils already carrying the rule.
- F5: the resume-line rationale cites the compaction re-render and the recording-not-enabled case; the old Open risks bullet 1 is gone.
- F6: D4 is append-only, and appending nothing leaves the prompt byte-identical.
- F7: the `plan.go` comment is in Scope's docs bullet and the docs criterion, and the notes bullet is gone.
- F8: the duplicate Auto-mode bullet is gone, and the single restarted-run risk remains.
- F9: D2 forbids copying `00-overview.md` from the archive, and the old risk bullet is gone.
- F10: D6 says "every session `claudeengine` spawns", with no enumerated list.

`support-log.md` interview entries 8 and 9 match the revised D6 and D7.

## Residual observation, not a finding

The first `claudeengine` acceptance criterion ties each notice sentence to "that deny's `PreToolUse` entry".
Interactive runs install an AskUserQuestion `PreToolUse` entry that records rather than denies.
D6's own bullet already limits the announcement to autonomous runs with the deny on, so the criterion reads unambiguously in context, and this does not block.

## Goal

```markdown
Imported from [issue #275](https://github.com/Knatte18/loomyard/issues/275).

**What happened.**
An implementer ran `lym gate test` in the foreground with the Bash tool timeout set to 3600000 ms, as the gate rule says.
Several times the harness moved the call to the background after 180 s.
Each time the call then ended with exit 75 and had to be run again.

**Why it is abnormal.**
The gate rule requires foreground calls.
Exit 75 with `done: false` is the documented keep-warm answer,
but it is meant to return to a foreground call.
The `keep_warm_cap` default in the claude engine's `template.yaml` is 250s, longer than the 180 s at which the call was backgrounded
(the value in effect for this run was not recorded).
The launch line sets `BASH_MAX_TIMEOUT_MS` to one hour, and the engine's docs say no Bash call stays in the foreground past that ceiling;
these calls left the foreground far earlier.

**What to do.**
Find which harness setting backgrounds a foreground Bash call at 180 s.
Then either set it in the launch and resume lines, or keep the keep-warm cap below it,
so a gate call always answers while still in the foreground.

Provenance: task `dobby-run-watch-build`, note `04-run-watch-wiring.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - subagent-ttl-scope-understated (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

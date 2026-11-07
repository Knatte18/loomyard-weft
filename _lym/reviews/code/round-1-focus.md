---
round: 1
focus:
  - "internal/shuttleengine/send.go `pasteVerified` and `clearBoxBeforeCommand`: does the clear (Claude's C-u) run only after the idle guard read the box empty and the operator-activity hold passed, and can it wipe operator text typed between that read and the clear?"
  - "internal/shuttleengine/send.go `confirmPasted` with `startsNoTurn`: can a turn start past the paste come from something other than the command taken as a prompt (a queued inject, a hook, a subagent turn), so a command that ran is failed with `ErrCommandTakenAsPrompt` and requeued, typing it twice?"
  - "internal/shuttleengine/send.go `retryLanded` and the retry paths: does a retry of a command send after `ErrCommandTakenAsPrompt` or a failed clear keep the pending-send record consistent, and does the clear replay on each retry?"
  - "internal/creelcli/delivery.go `mapDeliveryError`: does `ErrCommandTakenAsPrompt` really fall to the requeue path, and could a requeue loop forever when the leading whitespace is not clearable by the engine's clear?"
  - "internal/shuttleengine/claudeengine/command.go `OpenLaunch`: does `sh.Chain(exportTTL, ...)` give a correct line in both shell dialects (POSIX and pwsh) for the resume-or-fresh fallback, and does the TTL export reach the claude process in both?"
  - "internal/shuttleengine/claudeengine/command.go `OpenLaunch` against the decision record's 'any other window launched through the open path': is the claim that `Runner.Open` has one caller true, and does an unmapped or empty `spec.Role` resolve to the same TTL a run gets?"
  - "Test economy in internal/shuttleengine/claudeengine/command_test.go and internal/shuttleengine/session_test.go: do the new rows duplicate existing coverage, and is every behavior the diff adds (clear before paste, taken-as-prompt failure, TTL export, invalid-TTL refusal) covered?"
  - "Decision record Goal, part 1 'find where the space came from': does the diff or change description claim a cause the evidence does not support, given the record calls the source unproved?"
---

Seed focus for round 1.
The measuring stick is the decision record's `## Goal` alone; it has no plan and no `Added after Discussion` entries.
The change description's claims (53 successful `/compact` sends firing no `UserPromptSubmit`, a single `Runner.Open` caller) are claims to check against the diff.

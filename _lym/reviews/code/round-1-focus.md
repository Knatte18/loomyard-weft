---
round: 1
focus:
  - "internal/adoptcli/check.go: does the check set the mark at every record change (start, each tier start, each tier end, end) in order, with the D2 texts for starting, in-flight, between tiers, green and failed, and does a failing mark leave the record and the notices unchanged?"
  - "internal/adoptcli/adopt.go and cli.go: does the envelope stay one JSON line, with `message` set only when the check started or was found running, `suite_warning` kept and no message on a failed start, and the tiers omitted when the gate config does not read or has no `adopt` key?"
  - "internal/adoptcli/deps.go: is the tiers read from the gate config carried on `suiteStart` as D1 states, and does `hubCheckDeps` fill the mark seam with a reed engine over the prime?"
  - "internal/reedengine/checkmark.go and bar.go: does `SetCheckMark` set the three options, or unset them for an empty text, under the op lock and only when the session is up; does line 2 of the bar render the mark right-aligned before the clock, with the elapsed seconds only while `@lym_check_start` is set and the state styling the decision gives?"
  - "Does any Go code read the new `@lym_check*` options to decide anything, against `PATTERN-wait-mark-display-only`, or add a wait loop or process against `PATTERN-daemon-wakeups` and `PATTERN-spawn-observability`?"
  - "Docs landing: do docs/overview.md, the adoptcli doc.go, the reedengine doc.go and the help's step 8 name the status bar and the new reed mark as the repository's rules for this kind of change require?"
  - "Tests: do the new adoptcli and reedengine tests fake tmux and the check's seams as `PATTERN-test-speed` and `PATTERN-test-isolation` require, cover each new behavior, and avoid duplicating an existing test's coverage?"
---

Seed focus for round 1, drawn from the board entry's design and the decision record's D1 to D3.

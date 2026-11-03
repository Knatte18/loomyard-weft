## Goal

```markdown
## Seen (2026-11-02)

In run `finalize-fix-outside-lock`, the Code-Burler-Fix fixer finished its turn.
Claude Code then showed its "We recommend compacting earlier / autocompact at 400k?" picker in the shuttle session, where it replaced the input box.
The Code-Burler-Fix verify gate found one fixed-column comment wrap and tried to re-prompt the fixer.
The trace logged "re-prompt not delivered, retrying on a later tick without spending an attempt … no input box is on screen; a menu or picker" once a minute for over 30 minutes.
The re-prompt spends no attempt and never escalates, so the run waits forever; only dobby's quiet notice surfaced it.

## Workaround

A single Escape key in the fixer's pane closes the picker ("esc not now"); the gate's next tick then delivers the re-prompt.

## Expected

- Shuttle dismisses known mid-session pickers the way `claudeengine`'s startup dismisses the one-time trust gate: when readiness holds a pane with no input box, it matches the picker by its wording (here "We recommend compacting earlier" and "esc not now") and presses Escape, which keeps the current setting.
  Today the trust gate is handled only at startup, and readiness only reports "no input box is on screen; a menu or picker may be open".
- A picker shuttle does not recognise still ends the wait: the re-prompt escalates after a bounded number of ticks instead of retrying without end.
- No Claude Code setting is documented to suppress this picker (operator, 2026-11-02: it is rare, so shuttle handles it rather than a settings change).
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - dismiss-picker-escape-outside-pane-text-injection (class design, severity BLOCKING, open in rounds 1)
  - dismiss-picker-running-turn-guard-untested (class scope, severity MEDIUM, open in rounds 1)
  - no-box-reprompts-reset-semantics-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

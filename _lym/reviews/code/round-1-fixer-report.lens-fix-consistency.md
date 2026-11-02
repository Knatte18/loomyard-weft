```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix change set for stale siblings or unmatched twins (lens fix-consistency).

- The F1 edit in `PATTERN-parent-directive.md` agrees with the code as the review describes it. The `mode_rules` marker is what `VerifyTable` fills, and the line about the orch exemption and the test over the stencils still stand.
- The F4 header edit in `seats.go` matches `VerifyTable`'s doc comment, which already describes the `cfg.VerifyInteractive` behaviour.
- The F6 judging-text edit restores the old autonomous wording. Its twin, the stencil opening line "one session", stays as it is. That line is shared with the operator mode, so it cannot say "autonomous". The change description's claim that the autonomous text renders as before is still false for that line. Correcting it belongs in `_lym/landing/summary.md`, outside the fixer's write surface, so the fixer-report should record it.
- The F3 edit changes where the run counter's mode comes from. The `verify_test.go` fake `VerifyTable` now sets `Interactive` from the config, so the operator-session test at the old line 772 still holds.
- My grep found no remaining "non-interactive" claim in the drift packages, `pattern/` or the drift stencils.

Further tool calls:
1. One `grep -rniE` with a follow-up `grep -rn` over `internal/driftengine`, `internal/driftcli`, `pattern`, `PATTERN.md` and `contracts/stencils/drift`, looking for stale "non-interactive" or "discussion role" wording. My context did not hold the contents of those places.

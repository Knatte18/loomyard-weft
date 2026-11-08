## Goal

```markdown
## Seen (operator, 2026-11-08)

When a Code-Review round runs a Codex assistant reviewer beside the Claude reviewer, reed puts the Codex pane in a column of its own beside the Claude reviewer's pane, and that column keeps a fixed width: it does not follow when the window or the Claude pane is resized.

## Idea

The two reviewer panes share their space evenly and scale together: each takes half the width, on every layout render and every resize, as other side-by-side strands do.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-08): The pane border itself stays put on a window resize

Decision: Observed on Linux: resizing the whole terminal window shrank the Claude reviewer pane while the Codex assist pane kept its width; the border between them moved, the Codex pane did not scale. Both panes must change width together, half each. The fix is in reed's layout handling (for example re-applying even-horizontal on a client or window resize, or a hook that does it), not codexengine's --no-alt-screen, which stays as it is.

Rationale: Operator, 2026-11-08: the Claude pane got smaller while the Codex pane stayed as wide as before.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - os-bound-subtest-registered-on-windows (class consistency, severity BLOCKING, open in rounds 1)
  - resize-hook-zoom-skip-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

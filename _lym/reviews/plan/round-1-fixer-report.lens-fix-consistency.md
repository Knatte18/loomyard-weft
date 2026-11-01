```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fixes still agree with the plan cards, decisions and cross-references around them (fix-consistency lens).

- LOW — the `**ImpactSummary:**` of card 11 (`_lym/plan/11-adopt-holds.md`) is now stale. It still says the change stays "all inside adoptcli and its tests". The fix added `internal/beamcli/doc.go` to card 11's Edit list, with an Intent line that rewrites its `lym adopt` sentence. So the summary no longer covers what the card edits.
- LOW — the `**ImpactSummary:**` of card 12 (`_lym/plan/12-check-green-end.md`) is now stale. It still says the green end works "inside adoptcli over existing tender, batten, orch and fronttask calls". The fix added `internal/landingshed#ownerHaltReason` and `internal/battenshed/doc.go` to its Edit list and gave both a new Intent line. Neither package is in the summary.
- LOW — the `**ImpactSummary:**` of card 13 (`_lym/plan/13-check-fix-loop.md`) is now stale. It names the loop's inputs as "fronttask, the tries file and fabric's branch listing". The fix added a `Known` seam over `boardengine.LoadConfig` and `Vocabulary.Known`, and made the `gate` label depend on it. The board vocabulary read is missing from the summary.

Everything else I checked agrees:
- Card 4's end-only error lines up with card 6's step 1 and its `TestRemove` row, card 6's `HandRemoval` text, card 12 step 5, card 13 case 4, and D8 ("fails the step", the cancel named first).
- Card 11's D2 warning text matches D2 and its acceptance criterion in the decision record.
- Card 13's gate-label rule matches the parent's recorded decision.
- Card 13's new "starting" state does not conflict with card 14's states, which read an unstarted fix task as having no row.

Out of scope for this lens, noted for fix-claims: card 6's new Bound says dobby "exits at its owner read before it ticks". `internal/dobbyengine/doc.go:39` says dobby reads the owner at start and on each probe tick. So the gap is up to one probe interval, not only a race at that read.

Tool calls:
- Read `_lym/plan/00-overview.md` and `_lym/plan/14-adopt-run-report.md`. My context did not hold the overview's card index or card 14, which reads `fixRunHalted` and the fix task's states.

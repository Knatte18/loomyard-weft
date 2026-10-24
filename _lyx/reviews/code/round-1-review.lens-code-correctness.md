```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff (`cardHeadroomFindings`, `CardNamesPatternEntry`, the `patternPlanFindings` edits). I traced each by hand from the diff and my inherited context, with no tool calls.

**List**

- `cardHeadroomFindings`, `cards` empty, `removed` non-empty: every ID has `IndexFunc` of -1 and goes to `removedUpfront`. The loop does not run and the result is nil. Correct.
- `cardHeadroomFindings`, `removed` nil: both maps stay empty. `running` is `ProjectOverview(overview, nil, nil)`, which is the overview unchanged, so only each card's own growth is measured. Correct.
- `cardHeadroomFindings`, earlier card grows past the line and a later card shrinks it: `running` is under the line, the earlier card yields `next` over `HeadroomBytes` and larger than `running`, so it is flagged. The later card shrinks, so `Headroom` is false. One finding on the earlier card, as the acceptance criterion asks. Correct.
- `cardHeadroomFindings`, boundary `len(next) == HeadroomBytes` or `len(next) == len(running)`: `pattern.Headroom` returns false when `len(head) <= len(base) || len(head) <= HeadroomBytes`. This is the same rule the card gate uses. Correct.
- `cardHeadroomFindings`, card names removed entry B and fences a growing line: `ProjectOverview` applies prescribed lines first, then the removals. `next` is net of both, so a card whose net change is no growth passes. Correct.
- `cardHeadroomFindings`, removal named only by a later card: it is credited to that later card, so the earlier growing card is flagged. Correct.
- `cardHeadroomFindings`, removal named by no card: `ProjectOverview` drops it before the first card, so `running` starts smaller. Correct.
- `cardHeadroomFindings`, removal naming a card that already fences a line for the same entry: `pattern-removes-prescribed` already flags this, and the projection stays well-defined (fence applied, then removal). Correct.
- `cardHeadroomFindings`, a removed ID the worktree overview no longer holds, for example because a done card removed it: `findBullet` returns not found and the removal is a no-op. Correct.
- `cardHeadroomFindings`, done cards: `cards` holds only pending cards, so they are never walked or flagged. `running` starts from the worktree overview, which already holds their work. Correct.
- `cardHeadroomFindings`, duplicate IDs in `removed`: both are credited to the same card, and the second removal finds nothing. Harmless.
- `cardHeadroomFindings`, finding fields: `Card` is `card.ID()`, severity is blocking, and the detail names the card number and the way forward. Correct.
- `patternPlanFindings`, whole-plan `pattern-headroom`: the detail is `finding.Message + "; way forward: " + patternHeadroomWayForward`. The constant text is character-for-character the old literal, so the output is unchanged. Correct.
- `patternPlanFindings`, early returns (nil `PatternBase`, no `PATTERN.md`, `PatternBase` error): unchanged, and the new call sits after them, so it runs only when both overviews were read. Correct.
- `patternPlanFindings`, check order: `cardHeadroomFindings` is appended after `pattern-headroom` and before `pattern-line-unprescribed`, matching the updated doc comment. Correct.
- `CardNamesPatternEntry`, `` `PATTERN-ab` `` against id `PATTERN-a`: the match string includes the closing backtick, so there is no prefix match. Correct.
- `CardNamesPatternEntry`, bare ID without backticks: not matched, so the removal falls back to the upfront credit. Correct.
- `CardNamesPatternEntry`, empty `id`: the match string becomes two backticks. It cannot be reached, because the caller skips changes with an empty ID.
- `CardNamesPatternEntry`, a card that mentions the ID without removing it, such as a cross-reference in a fenced line: the removal is credited to that card, earlier than the true removing card. The check can only under-flag here, never invent a failure, which the record's Open risks accepts. This is not a defect.

**Findings**

None.

**Tool calls**

None made.

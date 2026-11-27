```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the F12 change set against its siblings and cross-references.

1. **LOW**: `_lym/landing/summary.md`, "Design decisions", the rewritten watched-guard bullet.
   - The rewrite reversed the reasoning. It now reads "stays at the task gates, so it is no whole-repository check".
   - The old text ran the other way: it reads a bounded set of files, so it is no whole-repository check, and that is why it stays.
   - Staying at the task gates is the decision, not the reason a guard is not a whole-repository check.
   - A wording that holds: "A watched guard that reads a bounded set of files is no whole-repository check, so it stays at the task gates: a narrowed gate runs it only when one of those files changes, and a scope-all gate runs it in full."

I found no other disagreement. These siblings still agree with the change:
- `PATTERN-test-speed.md`: the gate-keys table, the guard-section exceptions, the `//lym:only` section and the `//lym:watch` section
- `PATTERN-test-only-code.md`
- the `upkeep` comments in `internal/gateslot`
- the board-tidy upkeep stencils
- the summary's "tier-1 tests" note on the hub's `upkeep` key

My two searches found no other text claiming the C# fixture compile runs at task gates.

Tool calls:
- grep for "whole gate", "always runs" and "watched test" under `internal`, `cmd`, `contracts`, `docs/reference` and `pattern`: my context did not show other copies of the dropped "a whole gate always runs it" claim.
- grep for "C# fixture", "dotnet build", ".NET project" and "upkeep" under `pattern`, `contracts`, `internal/gateslot` and `internal/planglyph`: my context did not show the cross-references to the fixture compile or the `upkeep` key.

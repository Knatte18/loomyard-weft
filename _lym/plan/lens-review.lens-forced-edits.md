```yaml
lens: forced-edits
outcome: reported
```

Task: forced-edits lens review of cards 1–9 and the overview.

**LOW — the `constructor` type isn't listed on card 5 (`05-engines-with-seams`).**
- The Intent says "The registry's `constructor` type takes the seams as a third parameter", which changes that named function type's signature.
- The card's `**Edit:**` list names `internal/shuttleengines#registry` and the constructor's call sites, but not `internal/shuttleengines#constructor`.
- If `constructor` is a declared type in `internal/shuttleengines` (the Intent names it as one), the card should list it with a re-sign arrow, for example `` `internal/shuttleengines#constructor` -&gt; `type constructor func(cfg shuttleengine.Config, dirs configengine.Dirs, seams LaunchSeams) (shuttleengine.Engine, error)` ``, with the result types copied from the current declaration.
- I could not confirm the declaration within my call budget. Run `lym quarry resolve internal/shuttleengines#constructor` before acting on this.

No other forced edit is missing from the cards.

Calls made:
1. Grepped callers of `Passages`, `StandingPassages`, `PassageReader`, `passagesOf`, `passageItems` and `skillListing`, plus the docs that mention them. The pre-read had resolved the glyphs but not their callers.
2. Listed the test functions in `internal/heddleengine/render_test.go` and grepped the PATTERN, HEDDLE and overview docs and the run-root scans. This placed the `r.Passages` calls at lines 754–793 inside `TestRender`, which card 1 lists.
3. Read the guard lines of each other run-root scan. This tested card 2's claim that they already skip a directory with no `run.json` silently.
4. Read `loadRunState`, `agentwait.go:42-54` and any engine manifest code. This confirmed that a missing `run.json` reads as not found, with no warning.
5. Read `state.ReadJSON`'s lock handling and `PATTERN-lym-files` on naming. This checked whether reading a `manifests` directory creates a lock file and whether the new directory name breaks the naming rule. It does neither.
6. Grepped the new handle names for collisions, `lymtemp.MkdirTemp`, and every use of `heddle passage`. This confirmed no name collides, `MkdirTemp` exists at `internal/lymtemp/lymtemp.go:39`, and the only assertion of the old item name is `codexengine/prepare_test.go:415`, inside a test card 1 lists.

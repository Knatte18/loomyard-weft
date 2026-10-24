```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (fix-proposal lens).

1. **MEDIUM**: F9's proposal contradicts a line the fix left untouched in Decision 4.
   - Changed text: Decision 4's `{filter}` bullets, now keyed on the tier's share, and the matching acceptance criterion.
   - Case: a narrowed `dotnet` call lists a tier whose command lacks `{targets}`, such as `dotnet test Models.sln {filter} --no-build`.
   - That tier's share is narrowed, so the new second bullet renders the composed expression `(<tier filter>)&(<terms>)`.
   - The untouched line "A tier whose command lacks `{targets}` runs once, whole, with its tier filter only" renders the tier filter alone.
   - The over-budget fallback ("falls back to that project's whole invocation, with the tier's own filter only") is a second narrowed-share rendering the new bullets leave out.
   - Evidence: Decision 4, the bullets beginning "a tier whose share is narrowed by the `dotnet` scoper" against the paragraph after them.

2. **MEDIUM**: F8's per-tier rule reads as one build per opted-in tier, against Decision 6's one build per call.
   - Changed text: Decision 3's empty-test-scope paragraph ("a tier that opts in with `shared_build` runs the build step (Decision 6) and no test command") and the criterion "runs, for each listed tier that opts in with `shared_build`, the build step and no test command, inside that tier's slots".
   - Case: two listed tiers, tier0 and tier1, both opt in, and the closure selects no class.
   - The criterion says the build runs inside each of their slot holds.
   - Decision 6 runs it once, "inside the slot hold of the first opted-in tier in plan order".
   - Nothing says whether the second opted-in tier acquires slots at all or runs nothing.
   - Evidence: Decision 6's first bullet against the rewritten Decision 3 paragraph and criterion.

3. **MEDIUM**: F1's proposal leaves a card whose Targets are all inert with no stated scope.
   - Changed text: Decision 3's `--card` bullet ("a Target whose file is inert (Decision 5) is dropped from the seed") and the card criterion.
   - Case: a card whose only Target is `docs/dotnet-hub.md`, with a non-empty `inert_paths`.
   - Every Target is dropped, so the seed is empty.
   - Decision 2 defines the empty test scope only for "a non-empty seed", and defines the empty scope only for "an empty or inert-only diff".
   - Decision 5 says its rules "apply only to a diff", and its first bullet covers only "a computed or `--base` scope".
   - Nothing says whether this card runs no tier (the empty scope), runs the build (the empty test scope), or runs whole.
   - A `carry: false` tier's handling of it is unstated too.
   - Evidence: Decision 2's result bullet, Decision 5's lead paragraph and first bullet, Decision 3's empty-scope paragraphs.

4. **LOW**: F13's proposal makes the tier1 test leave processes behind.
   - Changed text: the tier1 criterion's new line, "The fixture holds at least two projects MSBuild builds in parallel, and the fixture's build command passes `-m`, so an out-of-process node starts."
   - With node reuse at its default, which the same criterion requires, the node it forces to start outlives the test until the SDK's idle timeout.
   - Decision 7 itself says idle nodes "serve later gate calls until the SDK's idle timeout".
   - No teardown is stated, such as `dotnet build-server shutdown` scoped to the fixture, or ending the nodes after the assertion.
   - Evidence: the testing rule "Teardown removes every file, socket and process the test created", which `PATTERN-test-isolation` in Constraints invokes.

Other proposals I challenged and found sound:
- F1's type-glyph seed: a nested type seeds its outer type, which over-selects safely.
- F2: `build` as an open map is as lax as `tiers` already is. `gateslot.LoadConfig` decodes with plain `yaml.Unmarshal`.
- F3, F6, F7, F12, F14 and F15 hold as proposed.
- F4: format specifiers inside an interpolation hole only over-select.
- F5: `+` is not special in VSTest's filter grammar, and a generic class's term still matches by `~` containment.
- F10: `.scratch/` is skipped only in the closure scan, and a changed file there still seeds.
- F11 was fixed differently from its proposal, so it is outside this lens.

Tool calls:
- Grepped `internal/gateslot/config.go` and `internal/configengine` for open-map handling, to test F2's claim that `configengine` carries `build`'s sub-keys whole. My context lacked the configengine side.
- Grepped `internal/gateslot/config.go` for its decode calls, to see whether an open map's sub-keys are validated strictly. My context lacked how `gateslot.LoadConfig` decodes.

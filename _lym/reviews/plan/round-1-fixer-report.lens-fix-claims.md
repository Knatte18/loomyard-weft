```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every claim the fix added or rewrote against the code at HEAD.

**LOW: card 4's line numbers for `pattern/PATTERN-test-speed.md` are off**
- Changed text: card 4's new list item for `pattern/PATTERN-test-speed.md`, "states the `//lym:seam` rule (lines 94-100), injected clocks (lines 102-106), `t.Parallel` by default (lines 108-110) and that a skip never names the OS alone (line 119)".
- Evidence from the file:
  - The `## Reflective tests carry //lym:seam` section is lines 94-102, with its bullets at 96-102.
  - `## Time is injectable` is lines 104-108.
  - `## Tests run in parallel` is lines 110-113.
  - Line 119 (the OS skip rule) is correct.
- The conclusion holds, because every rule is stated in the file. Only the cited spans are wrong.

**LOW: card 13's new Bound says the replayer checks an entry "through `validateWrite` alone"**
- Changed text: card 13's Bound, "the board's Bolt replayer admits an entry through `validateWrite` alone, never this rule".
- Evidence: the replayer also runs `checkRunLocks(r.origin, after, …)` at `internal/boardengine/replay.go:534` and `:560`.
- The claim that matters holds: `checkDiscussedPromotion` is not run on a replay. Only "alone" overstates. Card 14's existing Bound uses the same wording, so the two cards agree.

**Claims that hold**
- No import cycle (card 13): `boardengine` imports `fabricengine`, which imports `shedrun` (`internal/fabricengine/addanchor.go`), and `shedrun` does not import `boardengine`.
- `pattern/PATTERN-remote-refs-lock.md` lines 5, 10 and 11, and `pattern/PATTERN-shed-recipe-registry.md` lines 8, 22, 27 and 29, are correct.
- Teardown lock modes:
  - Shared holders are only the two push paths, `teardownlock.go:81` and `pushanchored.go:164`.
  - Everything else takes the lock exclusive, through `lockPairTeardown` (`teardownlock.go:114-124`) or the lock at `remoteteardown.go:316` and `364`, a write lock per `remoteteardown.go:423-425`.
  - The exclusive callers are `add.go:188`, `remoteteardown.go:247` and the cleanup sweep at `cleanupremotewarp.go:149`.
- Card 22's `SplicePart` sentence: card 11 says `SplicePart` errors on an unknown id, bullet or PATTERN entry. The new `decision-unknown` and `pattern-unknown` checks, with the existing `criterion-unknown` and `risk-unknown`, cover all of these.
- Card 25: `Deps.Notify` is `func(line string) error` (`internal/pawlengine/deps.go:51`). `pawlDeps` drops the posted result (`internal/loomcli/pawlwiring.go:46`). Fakes exist in `world_test.go:180`, `validPawlDeps` and `envkit.FullEnv`, and BeamAwait's caller is `awaitRun.notify`.
- Card 23: `arbiterRecipeDescription` is tested in `TestRecipeKeyedWiring` (`internal/loomcli/recipekeyed_test.go:65`), and the `loomcli/doc.go` paragraph is at line 25.

Tool calls:
- Read `PATTERN-test-speed.md` lines 92-120 and the numbered `PATTERN-remote-refs-lock.md`, to check card 4's line citations; my context had these files without line numbers.
- Grepped the teardown lock takers and their lock modes, to check card 4's shared/exclusive claim; my context lacked the callers of `lockPairTeardown` and the lock calls in `remoteteardown.go`.
- Grepped `boardengine`'s imports, card 11's `SplicePart` errors and `replay.go:455-470`, for the cycle, step 2 halt and Bound claims; card 11 and the replay add path were not in my context.
- Grepped which packages import `shedrun`, to confirm `boardengine` reaches it through another package; my context lacked `fabricengine`'s imports.

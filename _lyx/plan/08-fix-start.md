# Card 8 — fix-start

**Edit:**
- `internal/burlerengine/profile.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/handoff.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/doc.go`
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
Add a run-wide `fix_start` option choosing when a round's fixer starts (D5): `parallel`, the template default and today's behavior, or `after-review`.

- `burlerengine` declares `FixStart` with `FixStartParallel` (`parallel`) and `FixStartAfterReview` (`after-review`); `RunOpts.FixStart` carries it, the empty value meaning parallel so every existing caller is unchanged, and an unknown value is an error from `Run`.
- Under `after-review`, `Engine.Run` starts the reviewer alone, waits for it, runs `reviewReady`, and starts the fixer only once Go has accepted the review and written the ready marker; it then waits on the fixer and decides the round by `join`'s existing rules (fixer outcome, failing fixer gate, the review-bytes comparison).
  A reviewer that ends without an accepted review starts no fixer, and the round reports the reviewer's outcome or error as it does today when the reviewer fails.
  The fixer's prompt is unchanged, so its first `lyx burler await-review` returns at once.
  Restructure `join` in `handoff.go` only as far as both orders need to share one place for the round's failure rules; `Resume` and `ProbeRound` are unchanged, so a resumed round with one live half is stopped and respawned as today.
- `loomengine.Config` gains `FixStart string` (`fix_start`), `template.yaml` carries `fix_start: parallel` with a comment naming both values, and `LoadConfig` refuses any other value, naming the two legal values; `ReviewSettings` carries the resolved `burlerengine.FixStart`.
- `shedrecipe.Env` gains `FixStart burlerengine.FixStart`, and `burlerRoundEntry` sets `RunOpts.FixStart` from it, so the engine never reads `loom.yaml`; `loomcli`'s `wire` fills it from `ReviewSettings`.
- `refusal-spec.md` gains the config row for an unknown `fix_start` value; `docs/overview.md`'s loom config line lists the key; `burlerengine/doc.go` describes both start orders under how a round runs and ends.

Bound: `after-review` only delays the fixer's start; it skips no gate, and `PATTERN-review-round`'s reviewer-before-fixer ordering and ready-marker handoff are unchanged.
The option lives in `burlerengine` and `loomengine` only, never in `shuttleengine` (`PATTERN-shuttle-provider-seam`).

Tests:

- `engine_test.go`: under `after-review` the fixer's `StartGated` is called only after the review was accepted and the marker written, a reviewer that dies, times out or leaves an unparseable review starts no fixer, and the default and `parallel` keep both starts before either wait; add rows to the existing round-failure and marker-lifecycle tables.
- `config_test.go`: `fix_start` loads `parallel` from the template and refuses an unknown value naming both legal values.
- `entries_burler_test.go`: `Env.FixStart` reaches the round's `RunOpts`.
- `wiring_test.go`: a `loom.yaml` setting `fix_start: after-review` reaches `Env.FixStart`.

**ImpactSummary:** With the template default parallel every round starts both halves as today; after-review changes only the start order inside burlerengine's Run.

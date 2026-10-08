# Webster round 2 — fixer report

All three findings were validated against the code and fixed, one commit each.
`go test ./...` passes after the last fix, and `lyx loom lint-comments` reports ok.

## Fixed

- **F1** (`f50380394`) — `validateFanKeys` now loads `burler.yaml` lazily at the first set fan key, so both keys empty still never reads it.
  A read or parse error is now refused as `loom config key "<key>": <error>; fix burler.yaml, or set the key to empty to run the segment solo`.
  The test table checks that every `LoadConfig` refusal names its key, and this message now does.
  The message's way forward and the new `config: burler.yaml unreadable` row in `contracts/specs/refusal-spec.md` satisfy `PATTERN-refusal-way-forward` for this refusal, which the diff introduced.
  Both tests were extended with a row each, not new tests, as the finding asked.
  `TestLoadConfig_Loads` has "empty fan keys never read a malformed burler.yaml".
  `TestLoadConfig_Refuses` has "set discussion_fan with a malformed burler.yaml", which asserts the key, `burler: parse ` and the way forward.
  The Refuses row also proves that the shared fixture `malformedBurlerYAML` does fail to parse, so the Loads row would catch an unconditional read.
- **F2** (`f1864c5c0`) — `TestWriteRoundFacts_LensSection` has a third subtest.
  It writes an unparseable review beside a usage record and asserts that the fan line and the "Review unparseable" line render and that no per-origin table does.
- **F3** (`ad3d43525`) — `docs/overview.md`'s loom entry now lists `discussion_fan`/`plan_fan` and `fan_review` in its `loom.yaml` key list.

## Disputed

None.

## Deferred

None.

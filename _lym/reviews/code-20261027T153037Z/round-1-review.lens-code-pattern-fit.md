```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repo's PATTERN entries and CLAUDE.md rules (lens code-pattern-fit).

**Rules checked**

- `PATTERN-shed-recipe-registry`: held. Drift-Await is a recipe row keyed by the existing `DriftAwait` constructor, and its on/off switch is `entry_gate: drift.enabled`. The row takes no config key.
- `PATTERN-refusal-way-forward`: held. The `capture` key is refused through `configRejectUnknown` with `driftKeyWayForward`; the `BaselineCaptureKey` row pins it.
- `PATTERN-test-speed`: held. Removed modes' rows are dropped; tests parallel; no sleeps added.
- `PATTERN-config-layers`: held. No config layer or template changed.
- `PATTERN-stencil-ownership`, `PATTERN-sole-parsers`, `PATTERN-shed-driver`: not touched.
- `CLAUDE.md` "Docs land in the same commit": held. driftengine, shedrecipe, loomrecipe and driftcli doc.go, docs/overview.md and the recipe header are updated; grep finds no `WaitCapture`, `AwaitRow`, `capture: wait` or "wait mode".
- `CLAUDE.md` board-entry rule: held. No `_board` path touched.
- `CLAUDE.md` semantic line breaks: held.
- Declared PATTERN changes landed: none declared; no PATTERN change in the diff.
- Commit messages and departures: held. One commit, no `Pattern-Departure:` trailer, none needed.
- Comment conventions: held.

**Findings**

None.

**Tool calls:** none.

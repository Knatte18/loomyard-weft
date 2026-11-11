```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

List per change:
- The review range `2136b28b..HEAD` is empty: `git diff` and `git log` over it printed nothing, and `git rev-parse HEAD` returned `2136b28b17a02852e5b5ca206a2892df220b7aca`, the base itself.
- The diff changes no signature, output, key, message, file shape or behaviour. There is no affected set to find.
- Callers, tests, docs, stencils and fixtures: none are forced to change.
- Declared PATTERN changes: the directive declares none, and none landed.
- Commit messages and departures: the range has no commits and no `Pattern-Departure:` trailer.

Findings: none.

One note outside the lens. `_lym/landing/summary.md` says the comment above the test assertion at `internal/loomcli/smoke_test.go` still calls the text "the read gate's invalid-state refusal". It does not say that text now reaches the log through the stop's detail. This is the writer's own to-check item, and it concerns the handler's summary-claim check, not this lens.

Tool calls: none; the manifest already held everything my procedure needed.

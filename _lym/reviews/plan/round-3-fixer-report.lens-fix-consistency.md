```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix set in cards 5, 6 and 8 agrees with sibling cards and cross-references.

- LOW. Concerns card 6's new Intent sentence, "The package declares the reference engine's name, `claude`, and the compared engine's name, `codex`, once each as named constants, and `Compare`, the manifest keys of `PrepareEach`, card 8's `Report` and every test of the package use them."
  The sentence covers only `internal/shuttleparity`, so card 8's twin in another package keeps the literal.
  Card 8's `TestManifestCmd` (package `internal/shuttlecli`) still asserts "prints `ok` naming `codex`".
  Card 8 also keys its stub's `ManifestReport.Manifests`/`Unmet` maps by engine name.
  Card 6 does not say whether the constants are exported, so an implementer of card 8 can't tell whether `shuttlecli` may use them or must repeat the literal, which `PATTERN-code-conventions` forbids.
  Evidence: 08-shuttle-manifest-verb.md Tests, "a seat with an unmet Codex requirement and nothing missing prints `ok` naming `codex`".

All other cross-references agree:
- Card 8's deletion list matches the interim entries cards 5 and 6 add.
- Card 8 already lists `cmd/lym#testOnlyCodeAllowed` under Edit.
- Card 7's tests fall under "every test of the package".
- `TestSpecs_MatchTheStartedSpecs` targets `seatengine.Specs`, not `Seat.Specs`, so clearing `ModelChoice` does not touch it.
- `ModelChoice` is the embedded `shuttleengine.ModelChoice` with an `Engine` field, so `ModelChoice{Engine: engine}` is well-formed.

Tool calls:
1. Grepped cards 07, 09 and 00 for engine names, the allowlist, `TestMain` and `ModelChoice`. My context held none of those siblings.
2. Grepped card 07 for `PrepareEach`/`Compare` uses, because card 7 was not in context.
3. Read card 07 lines 20–69 to see how its producers and tests use engine names and `PrepareEach`.
4. Grepped `ModelChoice`'s definition and how `seatengine.Seat` embeds it, to check that `ModelChoice{Engine: engine}` is well-formed.

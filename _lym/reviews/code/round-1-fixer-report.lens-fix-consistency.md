```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the change set. I found no findings.

I checked these twins and siblings, and each agrees with the change:
- **`MeasureCall` doc text**: `gateslot/doc.go` and the `tiers.go` doc comment both describe the error return with no `--tier` fallback. The `darnengine` doc and `spec.go` comment say only "a form the hub's gate config can run", so nothing there is stale.
- **Search for stale wording**: a search for "falls back to --tier", "with both values" and "the session has" in the non-test sources and docs found none left over.
- **`results: trx` settings**: the new `results` field appears in the `Tier` struct, the YAML decode, the load refusals, the template comment, `gateslot/doc.go`, `gatecli/doc.go`, `docs/dotnet-hub.md` and the green fingerprint. `docs/overview.md` and the PATTERN files do not list vstest or TRX settings, so there is no twin to update there.
- **Wiring tests**: the changed darn wiring row in `wiring_test.go` matches the new `darnwiring.go` behaviour, which now returns the config-absent error.

One cosmetic point, not a finding: `runTierChecked` prints the env line to stderr even when the tier then returns a refusal, but it logs the line only on a non-refusal result. A refused tier has no log, so this is consistent.

Further calls made:
1. Grep for stale `--tier 0` / `MeasureCall` / env-line wording across the non-test Go, markdown and yaml files, because my context held only the edited files.
2. Grep of `docs/overview.md` and `PATTERN-test-speed.md` for tier-setting lists, plus a `git show --stat` and a grep of `darnengine/doc.go`, because I needed to know whether twin docs enumerate tier keys or describe the call.

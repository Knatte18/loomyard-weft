```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the change set (F1–F5 fixes, including the `_lym/landing/summary.md` edits).

1. **LOW** — `_lym/landing/summary.md`, `## Design decisions`, first bullet (left unchanged while its twin in the intro was updated).
   - It still says "Deriving them leaves the key's tiers unused by the marked scope while every hub runs its marked tests."
   - The fix made that false: `markedScope` now reads `cfg.Gates[request.gate].Tiers` to decide whether a manual tier's marked tests are refused.
   - It also contradicts the intro sentence added this round: "The one exception is a tier declared `manual`: its marked tests run only when the key lists that tier".

2. **LOW** — `_lym/landing/summary.md`, `## Changes`.
   - The third bullet says "`internal/gatecli/doc.go` and `internal/gateslot/template.yaml` state the new rule", but `pattern/PATTERN-test-speed.md` now states it too and is not listed.
   - The section also omits the reworded undeclared-tier refusal in `internal/gatecli/tiers.go`.
   - The test bullet describes only the tier0 and tier1 run, not the added manual-tier refusal and listed-key cases.

I found no disagreement among `only.go`, `doc.go`, `template.yaml` and the PATTERN bullet: all four state the same manual-tier rule ("lists", not "covers").

Outside the change set, noted only: `doc.go:29` and the `manual` description on `template.yaml:10` still say a manual tier runs only when `--tier` names it, although a gate key listing it also runs it. That wording predates this fix.

Tool calls: none.

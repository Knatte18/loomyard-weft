```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the review's proposals that the fixer carried out.

**Findings**

1. **MEDIUM**: `internal/gatecli/only.go`, the new manual-tier loop in `markedScope` (F1 as proposed), and the matching sentences in `doc.go`, `template.yaml`, `PATTERN-test-speed.md` and `_lym/landing/summary.md`.
   - **Summary:** the refusal ignores `covers`, so a key that lists a tier covering the manual tier is now refused.
   - **Case:** `tier1` is `manual`, `tier2` declares `covers: [1]`, `upkeep` is `{tiers: [2], scope: marked}`, and a `tier1` test is marked `//lym:only upkeep`.
   - **Before the reviewed range:** the refusal applied only when no listed tier "is or covers" the test's tier (quoted in `_lym/landing/summary.md`), so this call ran the test.
   - **Now:** `slices.Contains(cfg.Gates[request.gate].Tiers, 1)` is false, so the whole call is refused, including its `tier0` marked tests.
   - **Why the key has opted in:** listing a tier that covers `tier1` means "its command runs anyway". `dropCovered` and `ranWhole` in `tiers.go` both treat a covering tier that way.
   - **Reach:** this needs a manual tier that another tier covers, which is an unusual config.

No other findings:
- **F2:** `Slots == 0` against `tier0`'s 1 does tell the two tiers' settings apart.
- **F3:** nothing breaks.
- **F4:** the new wording holds for both `--guards` and the marked scope.

Outside this lens: `template.yaml` line 10 and `doc.go` line 29 still say a manual tier runs only when `--tier` names it. A `--gate` key that lists the tier is another way, and that wording predates the fix.

Further tool calls: none.

```yaml
lens: fix-consistency
outcome: reported
```

Scope: whether the fixes to cards 1 and 3 agree with the sibling cards, the overview and their cross-references.

No findings.

- **F1 (card 1):** the field-drop sentence and the `testMapRun.prepare` paragraph now agree. Both are covered by existing Edit targets. No other card mentions `inert` or `Inert`: card 4's dirty-check work names neither, so nothing elsewhere depends on the dropped field.
- **F2 (card 3):** both the `mapSideBySide` bullet and its `TestMapSideBySide` row were changed. Nowhere under `_lym` outside the reviews folder is `runtime.NumCPU()` or a below-1 fallback still mentioned. The overview line for card 3 only says "within its parallel and memory bounds", which still holds.

Tool calls:
- Listed `_lym/plan` and searched `_lym` markdown for `NumCPU`, "below 1 means" and `inert`. My context didn't include cards 0, 2, 4 and 5 or the decision record.
- Searched the sibling cards for parallel and inert references. My context didn't include their text.

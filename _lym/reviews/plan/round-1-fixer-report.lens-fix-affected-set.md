```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set-based findings' searches and check that the fix covers every member of each set.

Unchecked: F2. Its set is the subtests that need narrowing, and the review names all of them; I ran no extra search because the fixture's subtests were already in context. F11 and F12 rest on a single site each, not on a set.

Checked:
- **F1:** covered. Rerunning the search over `TestGateTestMapLinkedPackages` finds that the surviving subtests reading `linked` or `linked/inner` are exactly the ones the fix names: `TheMapRecords…`, `rebuiltInnerStamp` with both digest subtests, `AGoEnvFailureEndsTheWalk` and `ADirtyLinkedPackage…`. The other matches are subtests card 1 deletes, or ones that only land files under `linked` without reading its map.
- **F7:** covered. The deleted planning files call these testcover symbols that nothing else uses: `PlanUpkeep`, `NewSelector`, `Selector`, `SelectorConfig`, `Target`, the `Upkeep*` types, `MissReason`, plus `Map.FailedAt` and `Toolchain.GoRoot`. Card 1's allowlist sentence ("PlanUpkeep, NewSelector with the Selector methods, Map.FailedAt, Toolchain.GoRoot and the helpers only they reach") covers the functions among them. The gatecli helper `ranNames` keeps a live caller at `testmappackage.go:753`, so card 1's deletes orphan no gatecli helper.
- **F10:** covered. The only non-research mentions of `GoRoot` are `internal/testcover/doc.go:68`, `runner.go:132-134`, which card 2 deletes, and `testmapplan.go:133/146`, which card 1 deletes.
- **F14:** covered. Selector mentions in non-research code are the header at `internal/gatecli/testmapstamp_test.go:1`, now in card 1's Intent; the `stampEnvironment` comment at `testmappackage.go:919`, which card 1 already drops; and `testmap.go:91`, inside the testmap verb's Long text, which card 1 deletes. Every other match is configengine's unrelated "selector" or the research docs.
- **F8:** covered. Among card 1's subtests, only `AMapRunReplacesAnExistingMapWhole` and `AnEllipsisTargetMapsEveryPackageBelowIt` reach a failing package (outcomes, alpha's hang, broken), and card 3 now lists both. `ACardWithNoPlan` already wants exit 1.

no findings

Tool calls:
1. Grepped `TestGateTestMapLinkedPackages` for subtests reading the `linked` maps (F1's set); my context held only lines 1336-1460 of that test.
2. Counted the testcover symbols the deleted planning files use against all other uses (F7's set); my context lacked those call sites.
3. Grepped callers of `ranNames`, `CoveredKeys`, `CoveringTests`, `FailedAt` and `GoRoot()` for helpers card 1 might orphan (F7); not in context.
4. Grepped the repo for `GoRoot` and selector mentions (F10 and F14's sets); not in context.
5. Read `testmappackage.go:915-924` to confirm that line 919 sits in `stampEnvironment`'s comment, and checked the p0 fixture (F14 and F8); not in context.

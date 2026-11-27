```yaml
lens: fix-consistency
outcome: reported
```

Fork lens fix-consistency: checking that the round-3 changes to cards 14 and 15 agree with sibling cards, the overview and the decision record.

**LOW** — card 14, test preamble ("Every `gitexec` or `gitrepo` error they use is provoked in the real module…") against the unchanged line "Each row asserts the returned snapshot's phases, outcomes and read failures … and the error, as literals."
- **Summary:** the transport read-failure rows now take their text from a real `gitexec.Run` `ls-remote` error, but the literal-want sentence was left as it was.
- **Evidence:** that error text comes from git and curl and can name the temp working directory. A literal want for it either pins environment-dependent text or must quote the provoked error. The card does not say which, so the changed preamble and the unchanged assertion rule no longer agree.

**LOW** — card 14 **Uses:** list.
- **Summary:** the tests now call `gitexec.Run` and `gitrepo`'s `PushRebaseFree`, and neither is in **Uses:**.
- **Evidence:** card 15's **Uses:** lists the existing symbols its code and tests reach, such as `internal/fabricengine#List` and `#lsRemoteHeads`. Card 14's lists only `fabrickit` and `internal/fabriccore/doc.go`.

No conflicts found elsewhere:
- **Tier change:** the overview's test list (lines 86–87) gives no tier for `TestRunRule` or `TestOutcomeClassOf`. Card 12's "the core has tier0 tests only (D6)" covers `internal/fabriccore`, not the engine. The decision record's D6, line 138, sets tier0 for the core alone, and its "lowest tier that can check it" allows tier1 once real git errors are needed.
- **Card 15:** `TestSeedWarpExclude_SeedsWarpOnlyOnce` is already in **Edit:**, so the added `SeedWarpExclude` call needs no new target. The line 136 and line 144 rewrites agree with the deletion list and the ImpactSummary.

Tool calls:
- grep of the overview and cards 12–13 for executor and D6 tier statements: my context had only the overview lines that mention card 14.
- grep for where D6 is defined: my context did not hold the decision record.
- grep of `_lym/discussion/decision-record.md` for its tier0 and executor lines: needed to see whether D6 requires the executor tests at tier0.

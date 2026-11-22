```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Lens fix-affected-set: for each finding whose premise is a set, rerun its search and check the fix covers every member.

Unchecked: none. The finding premises that rest on a set were these:
- F4: sites naming the `harden-live-copies` key.
- F6: other recipes' fixer profiles that set target instructions.
- F2, F3, F8: callers of `hardenPreflight` and `CampaignCheck`.

**F4** — the repo-wide grep for `live-copies`, `LiveCopies`, `CopiesGateKey` and `in.Copies`, run over every file type, returns no match. Only the two `CampaignCheck{` literals remain: `internal/loomcli/hardenwiring.go:325` and `internal/hardenshed/campaign_test.go:27`. Both are already updated and set no `Copies` field. The key is gone from the template, `goReadGates`, the gateslot test defaults, the PATTERN table, the `NewLiveGate` and `CheckCampaign` docs and the tests.

**F6** — the `target:` blocks of the arras, tapestry and motif fix rows set `paths` only, with no `instructions`. Rendering `Target.Instructions` in the fix prompt therefore changes no other recipe's fixer prompt. Only the hardener fix row sets instructions.

**F2, F3, F8** — `hardenPreflight` is called from `wireHarden` (`hardenwiring.go:377`) and from `TestHardenPreflight` (`hardenwiring_test.go:172`). Both pass the new `entry` argument. The `.Entry` grep hit only unrelated structs.

Tool calls (3):
1. A repo-wide grep for the key, constant, `in.Copies` and `CampaignCheck{` sites, plus a grep of `BurlerFix`/`HardenFix` rows for `target:` and `instructions:`. My context held neither search.
2. `sed` over the arras, tapestry and motif `target:` blocks, to see whether those blocks set `instructions`. The first grep showed only line numbers.
3. A grep for `RoundFacts` and `.Entry` callers outside the test files. My context held no caller list; the hits were unrelated.

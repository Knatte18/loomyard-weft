```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I re-ran each set-based finding's search (F1, F2, F5) and checked that the fix covers every member.

Unchecked findings: none. F3 and F4 each concern a single site, so they have no set to rerun.

- **MEDIUM**, F2. Changed text: the allowlist sentences in card 5's and card 6's Intent, and the deletion list in card 8's Intent. The fix leaves out `internal/shuttleengine.MissingItems` from card 2, which no main package reaches until card 8.
  Evidence: card 2 creates `MissingItems` (`02-manifest-record.md:54`), and its only production caller in the plan is card 6's `Compare` ("`Compare(engines, manifests)` returns `MissingItems` of the reference engine's manifest…", `06-seat-parity.md:89`). `Compare` is in `internal/shuttleparity`, which no main imports before card 8. So every batch gate ending after card 2 and before card 8 fails the `testOnlyCodeAllowed` reachability guard for it, the same failure F2 describes. No card adds its entry, and card 8's deletion list does not name it.
  `internal/shuttleengine.ReadManifest` may be in the same position. Its only production caller in the plan is `PrepareEach` (card 6). Card 2's `copyManifest` text says only that it "reads `&lt;runDir&gt;/manifest.json`", so the plan does not say whether `copyManifest` goes through `ReadManifest`. If it does not, `ReadManifest` needs the same interim entry.
- **LOW**, F5. Changed text: card 6's Intent sentence "…`Compare`, the manifest keys of `PrepareEach`, card 8's `Report` and every test of the package use them". That sentence covers only `internal/shuttleparity`, and it never says whether the constants are exported.
  Evidence: card 8's `TestManifestCmd` lives in `internal/shuttlecli`, and its assertion "prints `ok` naming `codex`" (`08-shuttle-manifest-verb.md:72`) is an engine-name literal in another package. Unless the constants are exported and named for that test, it falls outside the set the fix covers.
- F1: no finding. Across every card's Create group, `internal/shuttleparity` is the only new test package, and card 6 now creates its `TestMain`. Cards 7 and 8 add tests only to that package or to existing ones.

Tool calls:
1. Listed every card's Create group, because my context lacked cards 1 to 4, 7 and 9, which the F1 and F2 set searches need.
2. Grepped the plan for `ReadManifest`, `MissingItems`, `Omitted`, `FileEntries` and `testOnlyCodeAllowed`, because my context lacked those functions' production callers in the cards I had not read.
3. Read card 2's Intent span on `copyManifest`, because my context lacked whether `copyManifest` reaches `ReadManifest`.
4. Grepped cards 6 to 8 for engine-name literals, because my context lacked card 7, and F5's set spans cards 6 to 8.

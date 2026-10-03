---
round: 1
focus:
  - "Card 1 (testkit-root) against D1/D3: does the decision record license the kit-on-kit exemption to the importer rule (non-test files under internal/testkit/ may import other kits) and the envelope RequireOK/RequireErr pair as a second admitted assertion (cards 1, 37 and the kit-conventions Shared Decision), and does card 1's Intent state what the kit-on-kit exemption lets through and what bounds it?"
  - "Card 3 (gitkit-spawn-rule) against D4: D4 says the shared definition matches any gitkit. reference other than HermeticGitEnv, while card 3 matches only gitkit. followed by an exported identifier. Settle whether that narrowing is within D4 and, if it is a loosened guard, whether the Intent bounds what it lets through."
  - "Granularity and D12 package-by-package conversion: kit cards 19, 24 and 30 create the kit alone, while cards 18, 36, 37 and 39 create a kit and convert several packages in one card, and cards 6 and 28 convert four to seven packages each. Is each card one independently reviewable unit?"
  - "Test deletions in cards 12, 13, 16, 34 and 35: is each deleted test licensed by D7, D8 or #325 section 5 within D9's exception (help-prose tests, TestWatchdogLockPath_IsHubScratchDirLockFile, one of the two TestEnsureWatchdogSpawned early-return tests, the merge-conflict envelope integration test), and does card 11's one-engine-test-per-remedy-string goal square with the D9-protected livestate_refusal_selftest_test.go keeping its pins?"
  - "D11 docs: cards 19, 24 and 30 state their kit's package doc; do cards 18 (locationkit), 36 (logcapture), 37 (envelope) and 39 (lyxbin) carry the package doc D11 requires for each kit?"
  - "ImpactSummary on the production-touching Edit cards 8 (hubforge AddPair delegating to AddPairWith) and 11 (validateWorktreeSlug, resolveEffectiveWarpURL, CloneHub returning sentinel-matching error types): does each give a blast-radius conclusion covering callers outside tests, such as any errors.As or string matching on those errors?"
  - "Fabric Vocabulary constraint on kit non-test files: the kit-conventions Shared Decision says kits avoid bare fabric vocabulary. Do cards 18, 24 and 30, whose kits fill fabricengine, landingshed and websterengine fields, carry that constraint where it bites?"
  - "Cards 19 and 36 make concurrency assertions that must pass under go test -race, but neither card's Verify nor the plan-level verify: section runs -race. Settle whether those race-only assertions are ever exercised."
---

Generation 0: `_lyx/loom/rework/` holds no round, so the decision record alone is the answer key.

The build-tag claims in cards 4–7 ("every file here is already integration-tagged") were spot-checked against the tree and hold, including the files without an `_integration_test` suffix, so they are not a focus.
The `envkit` and `shedfake` import sets were checked against their consumers' dependency graphs, and no cycle shows, so those are not a focus either.
The verify section runs `go test` on both untagged and `integration` tiers and compiles `smoke`, which the rubric accepts.
The smoke-file cards (6, 21, 28, 39) carry their own smoke `Verify`.

```yaml
lens: repo-invariants
outcome: reported
```

My task: the repo-invariants lens on cards 1 to 15. For each card I checked three things: reflective seam tests and the tier tag of each new test file, doc text the card makes false, and whether the card can pass its own gate given what earlier cards leave behind.

**Per-card list**

- **Card 1**
  - Seam: held. The `//lym:seam internal/shuttleengine#Config` tests in loomcli, battencli and envkit reflect over func fields. The new int field adds nothing they check, and the card names them under Uses.
  - Docs: held (the shuttleengine doc is extended).
  - Order: held.
- **Card 2**
  - Seam: held. The `Engine` interface has three implementers (claudeengine `Claude`, shuttlefake `Engine`, `fakeEngine`), and the card covers all three.
  - Docs: held. The claudeengine and shuttleengines docs are listed.
  - Order: held. `ttlConfig` is edited. The hubreconcile and configreg fixtures assert keys only, and the 250s cap is valid against their lifetimes.
- **Card 3**
  - Seam: held (`allowedImports` gains `internal/agentname`).
  - Docs: held.
  - Order: held, since card 2 already supplies `shuttlefake.KeepWarmCaps`.
- **Card 4**
  - Docs: held. Card 4 rewrites `shuttleengine/doc.go:136` ("the bound of the gate's slot wait") and `docs/overview.md:419`.
  - Order: held, since it uses card 3.
- **Card 5**
  - Guard: held. `verifyRunAllowed` still matches `land.go` and `runner.go`, and both keep calling `verifyrun.Run(`.
  - Tier tag: held (the existing `tier1 &amp;&amp; !windows` file).
  - Docs: held (the cardverify doc is listed).
- **Card 6**
  - Tier tag: held (new file `tier1 &amp;&amp; !windows`).
  - Docs: held. The package comment is updated and the import set is unchanged.
  - `TestMain` helper: held.
- **Card 7**
  - Guard: held. `runner.go` keeps a `verifyrun.Run(` call, so the allowlist entry is still seen.
  - Docs: held. cardverify has no import allowlist test.
  - Order: held, since it uses card 6.
- **Card 8**: held. The runproc doc's role list is extended, and the leaf import test is untouched.
- **Card 9**: held. Fields are optional on read, and the report doc and gateslot doc are listed.
- **Card 10**
  - Tier tags: held. The tier0 rows run in-process and spawn nothing, and `TestGateRunner` is tier1.
  - Hidden verb: held. `cmd/lym/clitree_test.go:216` wants a hidden verb to carry `AudienceInternal`, which the card sets.
  - Wakeups list and background file: held.
- **Card 11**: **broken**, see finding 1.
  - Held: the only callers of `newCommand` are `Command` and `runGateHelper`, and `helperSlotWaitEnv` has no users the card misses.
  - Held: `gatecli/doc.go:17` and `:157`, `gateslot/doc.go:51` and `docs/overview.md:494` all sit in files the card lists.
- **Card 12**: held.
- **Card 13**: held. The `TestGateCardVerify` annotation subtest is covered.
- **Card 14**: **broken (doc)**, see finding 2.
- **Card 15**: held. The steer assertion at `rows_test.go:45` sits in `TestCheckTexts`, which the card lists. The shellguard doc does not quote the old steer.

**Findings**

1. Severity BLOCKING · class consistency · location: card 11 (`internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub` is not in its targets).
   - Card 11 cannot pass its own tier0 card gate. Its Texts section rewrites the test verb's help and drops the single slot-wait cap, so the help no longer says "for up to &lt;cap&gt;".
   - The help subtest "help names the shared keep-cache-warm cap as the slot wait" still asserts `"for up to " + shuttleengine.DefaultAwaitCap.String()`.
   - No card edits or removes that subtest, and it is that test file's only use of the `shuttleengine` import.
   - Evidence: `internal/gatecli/cli_test.go:59-66`; card 11's **Texts** paragraph and its Edit list.
   - Fix: add `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub` to card 11's Edit list. The help subtest is dropped or rewritten to assert the new capped-call text, and the import goes with it.

2. Severity LOW · class consistency · location: card 14 (`//contracts/specs/loom-plan-spec.md`).
   - The spec's `verify-gate-no-wait` check says "A call without it exits 75 on a busy slot instead of waiting for one".
   - After card 11 that is false: a call without a strand or under a zero cap waits for its slots with no cap, and only an agent's capped front answers 75.
   - Card 14 already targets this file, but its Intent rewrites only the card and batch gate sentences, so this rationale stays stale.
   - Evidence: `contracts/specs/loom-plan-spec.md:594-596`; card 11's **Choice (D2)** paragraph; card 14's Intent.
   - Fix: card 14's Intent also restates the check's reason. A Verify line runs from Go outside an agent hook, and without `--wait` its gate call takes the line's slot and runs in-process, not on its own slots.

Tool calls:
1. Grepped the `PromptCacheLifetime` implementers, the `newCommand` callers and the test users of `helperSlotWaitEnv`/`slotWait`, to check card 2's implementer set and card 11's forced edits; the manifest held only the declarations.
2. Grepped the `//lym:seam` markers and import allowlists for the touched packages, to check the seam and leaf invariants; the manifest did not include them.
3. Read the name of the test that owns `rows_test.go:45`, and grepped docs for the cap and slot-wait text, to see which card covers the steer assertion and which docs card 11 falsifies.
4. Grepped tests pinning `prompt_cache_ttl` outside claudeengine, internal-audience guard tests and await help assertions, to check card 2's template forced edits and card 10's hidden verb.
5. Read those `prompt_cache_ttl` assertions and the clitree hidden-verb rule, to see whether new claude.yaml keys or a hidden verb break them.
6. Grepped the docs naming `PromptCacheLifetime` and the gatecli doc's `testRequest`/phase text, to check doc text that cards 2 and 10 might falsify.

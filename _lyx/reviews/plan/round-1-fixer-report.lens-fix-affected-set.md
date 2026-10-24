```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the reader searches for the set-based findings and checked that the fix covers every member.

Findings left unchecked: F10, since the old-spelling sites were not searched again beyond what the parent's context already shows. I checked F4's callers only for `darnTreeScope`, not again for `guardRunOf`. I did not confirm whether a landingshed `fixedGateTierLists` exists or whether card 1 means `fixedGateTiers`, because I ran out of tool calls; validate-plan passing suggests the glyph resolves.

**MEDIUM — card 6 (landing-row-keys), the F11 sentence "the hand-built Deps, found by a search for `Deps{` … set `GateKey` … `TestVerifyGate_CheckMapsVerifyResult`, `newMergeInFixture` … `finalize` in `newTestDeps`"**
- **Summary:** The landing tests that assert the record kind get their gate from a hand-built `verifyGate` literal in `newGateFixture`, not from `Deps`. Setting `Deps.GateKey` therefore never reaches those gates, and `newGateFixture` is not on the card.
- **Evidence:**
  - `internal/landingshed/verifygate_test.go:103` builds `f.gate = verifyGate{command: …, tiers: fixedGateTiers, …}` directly, bypassing `newVerifyGate`.
  - `newGateFixture` supplies the gate to `mergein_test.go:102` (`newMergeInFixture`'s `gate:`), `finalize_verify_test.go:55` (`TestFinalizeVerify_Fail`'s fixture), `finalize_gate_test.go:42`, `finalize_merge_test.go:143`, and `verifygate_test.go:172/279/348/377` (`TestVerifyGate_CheckMapsVerifyResult` among them).
  - `TestVerifyGate_Clean` builds `verifyGate{}` at `verifygate_test.go:451`.
  - Card 6 has `recordFailure` write the key that "`verifyGate` keeps … in a new field that `newVerifyGate` fills". That field stays empty in these literals.
  - With the empty-key default gone, the card's own assertions fail: `TestMergeIn_PlanVerifyFailureRecord` expects kind `merge-in` and `TestFinalizeVerify_Fail` expects `finalize`.
  - The searched pattern `Deps{` does not match this set. The search needed is for `verifyGate{`.
- **Change:** add `internal/landingshed#newGateFixture` and `internal/landingshed#TestVerifyGate_Clean` under Edit. `newGateFixture` takes or sets the row key, with each caller passing `merge-in` or `finalize`.

Card 1's `Gates`/`TierList` set, card 7's `tree_scope_verify` set, card 15's `guard_tier` set and card 5's `darnTreeScope` callers: covered, with no further members found.

Tool calls:
1. The function around `gateslot/tiers.go:283` and the non-test `Gates` readers, to confirm card 1's set beyond the parent's grep.
2. `hubwide.go:455-461` and the symbols around every `tree_scope_verify` mention in darnengine and loomcli, to rerun card 7's set.
3. The exact `tree_scope_verify` lines in `darnengine/config.go`, to resolve comment lines the awk mapping assigned to the wrong function.
4. Every `verifyGate{` / `Deps{` literal and the users of `newTestDeps`/`newGateScratch`, to rerun F11's hand-built set beyond the parent's `Deps{` search.
5. `newGateFixture`'s body and its users, `premerge_test.go` and `guardedrows_test.go`, to see where the landing gates under test come from.
6. The function holding `darnwiring.go:38` and the users of `newGateFixture` and `fixedGateTiers`, to finish F4's caller check and F11's gate-fixture set.

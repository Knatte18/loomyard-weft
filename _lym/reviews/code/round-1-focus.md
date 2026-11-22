---
round: 1
focus:
  - "internal/gateslot/config.go LoadRefuseRaw and its claudeengine/codexengine callers: settle whether a launch that falls back past a failed full gate load still refuses on an unmergeable file or an invalid refuse_raw, and whether the warning reaches the operator."
  - "internal/gateslot/reinterpret.go RetireHardenLiveCopies and decodeGateKey: settle whether a scope-less entry is retired whatever its tiers, the way-forward flag stays correct when tiers differ, and the surviving constant at line 48 has a check that pins it."
  - "internal/configcli unsetKeys with configengine.Unset and yamlengine.RemoveKeys: settle whether --unset takes the same hub write guard, commit or fabric sync and module load check as --set, is refused beside --set, --print and --resolved, and removes a map entry or a whole key correctly, including a mapping the removal empties and an absent key."
  - "Docs landing: settle whether docs/overview.md, docs/shared-libs/configengine.md and the changed doc.go files match the diff's behavior, and whether the commit summary claims in _lym/landing/summary.md, including its Merge danger call, hold against the diff."
  - "Root-Cause trailers on 4ca9fdfb7 and 1feb3c612: settle whether each names the defect its diff removes."
  - "Test economy: TestDispatchSetProfiles is listed as contained by TestDispatchLayers; check its assertions against the covering test before filing."
---

Scope is the board entry's three ideas: a no-gate launch warns past a stale gate key, the harden-live-copies retirement covers a scope-less entry, and `lym config` gains a non-interactive key removal.
The decision record carries only the board entry, so the other sections are no measuring stick.

```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix for sibling documents or twins that no longer agree with the change.

**MEDIUM: stale module doc in `internal/driftengine/doc.go:4-6`**
- The changed text is the `ConfigSpec` shapes, plus the template and `ConfigSpec` doc lines that now say `--set` reaches one key inside a step.
- The package doc still describes `capture` and `compare` only as "each a command step written whole as `{command, dir, env, timeout_min}`".
- Line 6 names only the whole-step form, `--set 'capture={command: ..., timeout_min: N}'`.
- The sibling `internal/gateslot/doc.go` was extended in the original commit to add the one-key-at-a-time form (`--set tiers.tier0.env.LYM_TEST_TIER=0`, `--set build.timeout_min=10`) and the refusal of a key no step holds.
- The drift doc is the twin that was left as it was. The reviewer's fix key did not name it, but the module doc is where the behavior is stated.
- Line 6 also says only the template and the hub default layer set these keys. That stays true.

**No other twin disagrees:**
- `internal/configreg/configreg.go:133` passes `Spec: driftengine.ConfigSpec`, so the new `OpenMapShapes` flows through.
- `internal/configcli/doc.go:56` is generic, so leaving it unchanged is consistent with the fix.
- The `Step` error message in `internal/driftengine/config.go` still names the whole-step form, which is still valid.
- Both new template comment lines match the `ConfigSpec` doc.
- The shared `shapedTier` field set and the updated sorted key list in the refusal row agree: `command, covers, env, plain, retries, slots, timeout_min`.

Tool calls used (2):
1. `grep` for `capture={`, `capture.` and `OpenMapShapes`: to find other mentions of the drift `--set` form and other users of `OpenMapShapes`. My context had not shown the other docs.
2. `grep` in `driftengine/doc.go` and `driftcli`, plus callers of `ConfigSpec` and `ConfigOpenMaps`: to read the drift module doc and check `configreg` wiring. My context had not shown those files.

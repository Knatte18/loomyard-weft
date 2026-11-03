```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens — what the diff forces outside the lines it touches.

**Per change, the set found**

1. `Known` now lists each open map P as well as `P.<name>`.
   - Readers: the `unknown config key(s) ... (known: ...)` message in `internal/configengine/set.go:89,171`. It prints whatever `Known` holds, so it is correct.
   - Tests asserting `Known`: only `yamlengine/set_test.go` and `layers_test.go`, both updated in the diff. No outside test pins the exact list. Covered.
2. `unknownKeys` and `knownKeys` now accept P itself.
   - Every module with `OpenMaps` goes through `yamlengine.SetValues` or `SetSparse` via `configengine`'s `Set`: board (`types`, `labels`), darn, gate (`tiers`, `tier_parallel`, `build`), landing, reed, claude, loom, tapestry, drift.
   - For P holding a scalar or list (loom model keys, reed buttons), the scalar-or-list branch still runs first, so those are unchanged.
   - For P holding a mapping (`labels`, `types`, `tier_parallel`, `tiers`, `build`, `tree_scope`, the credit-engines map), `--set P={...}` was refused as unknown and is now a whole replace.
   - Docs: `configcli/doc.go`, `configengine/doc.go` and `yamlengine/doc.go` are updated, and `driftengine/config.go`'s `ConfigSpec` comment is updated.
   - `plugins/ly/skills/board/SKILL.md:51` only describes the per-name form `labels.<name>=` and stays correct. Covered.
3. The drift key description in `driftengine/template.yaml` already prescribes `capture={...}`, so it now matches. The Goal's alternative of describing the per-key form instead was not needed. Covered.
4. PATTERN entries.
   - `PATTERN-config-layers` is unaffected: `configengine` still owns the read/write, and no PATTERN entry is declared changed.
   - `PATTERN-refusal-way-forward`: the new refusal in `setWholeOpenMap` names the per-key form, and the unknown-key refusal names the held keys. Covered.
5. Commit message: it says "every declared open map P". The code does apply to every declared open map, and the shape refusal applies "under a declared shape". The claims match the diff.

**Findings**

1. Severity LOW, class `consistency`, location `internal/yamlengine/set.go:221` against `internal/yamlengine/layers.go:202-207, 258`.
   - The whole-map branch sits before the shaped-key branch in `SetValues`, but after it in `SetSparse`.
   - A path both in `openMaps` and under another shaped open map would route differently on the two paths.
   - No module declares nested open maps, except reed's `buttons.*`. That is a scalar-or-list case and is unaffected, so this is latent only.
   - Fix: put the branch in the same order in both functions.

2. Severity LOW, class `consistency`, location `internal/yamlengine/set.go:401`.
   - `mappingShapeError` returns nil for a mapping placed under a text or other single-value shape (`Fields` and `Entries` both nil), so `capture={command: {a: b}}` is accepted, whereas `capture.command=...` would write a string.
   - The file then fails later at `LoadConfig`.
   - This is a shape-check gap that the doc's "refuses a key its shape does not reach" does not call out.
   - Fix: refuse a mapping where the shape is `Text`, or document the limit.

I found nothing outside the diff that is left wrong by it.

**Tool calls**
1. Grep over tests for `known:`, `wantKnown` and `.Known` outside the two yamlengine test files, and over `.md` files for `<name>` open-map descriptions. I had no view of other tests or docs asserting the `Known` list.

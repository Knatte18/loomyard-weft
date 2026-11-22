```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each changed function and branch with concrete inputs.

**Per function or branch: the case I ran, and the result**
- `gateslot.LoadRefuseRaw`, success: a valid file returns `cfg.RefuseRaw`. Correct.
- `LoadRefuseRaw`, absent hub file: `ErrConfigAbsent` returns nil, nil. This matches the old engine behaviour. Correct.
- `LoadRefuseRaw`, scope-less `gates.harden-live-copies` (`decodeGates` refuses it) with a valid `refuse_raw`: `configengine.Load` succeeds, the unmarshal succeeds, `decodeRefuseRaw` passes, the warning is logged and the list returned. Correct.
- `LoadRefuseRaw`, a `refuseRetiredGateLists` failure: `LoadConfig` fails before the merge, `configengine.Load` still merges, and the launch goes on with the warning. This is the stated design ("any gate key but refuse_raw"). Correct.
- `LoadRefuseRaw`, a stack that does not parse or merge (`slots: [`): `configengine.Load` errors and loadErr is returned. Correct.
- `LoadRefuseRaw`, `refuse_raw: "go test"` (a scalar): the struct unmarshal errors and loadErr is returned, which carries the unmarshal type error. Correct.
- `LoadRefuseRaw`, an invalid `refuse_raw` entry plus a scope-less gate key: it returns loadErr, which names the gates key, not refuse_raw. **Finding F1.**
- `LoadRefuseRaw`, an invalid `refuse_raw` entry alone: loadErr is `decodeRefuseRaw`'s error naming refuse_raw. Correct.
- `claudeengine.Prepare` and `codexengine.Prepare`: an error wraps and refuses before any artifact, and the list reaches `cfg.RefuseRaw`, `installedGuards` and `writeLaunchManifest` alike. Correct.
- `RetireHardenLiveCopies`, scoped and equal: removed, with the "names the tier" rewrote line. Correct.
- `RetireHardenLiveCopies`, scope-less and equal (`harden-live` absent, so `[0]`; copies `{tiers:[0]}`): matches, so it is removed with the "names the tier" line. Correct.
- `RetireHardenLiveCopies`, scope-less with `[1]` against live `[0,1]`: removed, with the no-scope line on rewrote and the flagged line. Correct.
- `RetireHardenLiveCopies`, a copies entry that does not decode (`tiers: x`): the zero `GateKey` has scope "", so it is removed and flagged with tiers `[]`. Acceptable, since that entry also stops the load.
- `RetireHardenLiveCopies`, a copies entry with an invalid non-empty scope (`scope: coverage` or a typo): left and flagged, though `decodeGates` refuses it like a scope-less one. **Finding F3.**
- `RetireHardenLiveCopies`, scoped and differing: left, flagged, and the way forward names `--unset`. Correct.
- `RetireHardenLiveCopies`, a `harden-live` entry that does not decode: `liveTiers` is nil, so matches is false. Removed if scope-less, otherwise flagged naming `[]`. Correct.
- `decodeGateKey`: returns the zero `GateKey` on a decode error, and a null node decodes to zero. Correct.
- `yamlengine.RemoveKeys`, empty or comment-only input: the root is nil, every key is missing, and it errors with nothing written. Correct.
- `RemoveKeys` with `[present, absent]`: the in-memory removal is discarded, it errors and returns nil. Nothing is written. Correct.
- `RemoveKeys`, `labels.old` as the only entry: the child is emptied, so `labels` is dropped. Correct.
- `RemoveKeys`, `a.b` where `a` is a scalar: the child is not a mapping, so the key counts as missing. Correct.
- `RemoveKeys`, the last top-level key: returns `[]byte{}`. This is the same as `Prune` (layers.go:439-440), so no finding.
- `RemoveKeys`, the same key given twice: the first call removes it, and the second reports "no key X in the file" for a key the file held. **Finding F4.**
- `configengine.Unset`, a layer that does not apply: the `layerPath` error. Correct.
- `Unset`, an absent file: an error with the way forward, nothing written. Correct.
- `Unset`, a `RemoveKeys` error: wrapped with the path and the way forward. Correct.
- `configcli.unsetKeys` and `setPairs`: thin wrappers. Correct.
- `setChecked` with an unset: snapshot taken; before fails on the scope-less key; after loads; `newLoadFailure` is nil. Correct.
- `setChecked`, an unset that newly breaks the load (unsetting `build` while a tier sets `shared_build: true`): it rolls back, but the refusal reads "would not load with this value … way forward: --set the key to a value it takes". **Finding F2.**
- `setHubWide` with an unset that errors (a key the file lacks): rolled back, writeErr is set, and the result is the "…; path unchanged" error. The fake's commit count is 1 because the closure runs inside commit. Correct.
- `setModule` worktree path with an unset: the guard is skipped (correctly) for the worktree and local layers; it syncs once on success. Correct.
- `dispatch`: `--set` with `--unset`, `--print` with `--unset`, and `--unset` with no module each refuse with the right message before anything is read. Correct.
- `resolvedRefusal`: `--resolved` with `--unset` is refused. Correct.
- `Command` RunE: no args with only `--unset` reaches dispatch, which reports "module required with --unset", not help. Correct.

**Findings**

- **F1 · LOW · design · `internal/gateslot/config.go`, `LoadRefuseRaw`'s `decodeRefuseRaw` branch (the `return nil, loadErr` after `decodeRefuseRaw`)**
  - Summary: when refuse_raw is invalid and another key also fails, the refusal carries the full load's first error. `decodeConfig` checks `gates` before `refuse_raw`, so the launch refuses naming a gate key it says it never reads, and refuse_raw, the actual cause, goes unnamed.
  - Evidence: config.go decodeConfig, `decodeGates` (~line 365) runs before `decodeRefuseRaw` (~line 381); the diff's `LoadRefuseRaw` returns loadErr in that branch.
  - Fix: return `decodeRefuseRaw`'s own error, with the gate config way forward, in that branch.
- **F2 · LOW · design · `internal/configcli/configcli.go`, `newLoadFailure`, reached from `setChecked` and `setHubWide` on the `--unset` path**
  - Summary: an `--unset` that newly breaks the module load is refused with "would not load with this value" and "way forward: --set the key to a value it takes". An unset sets no value, so the way forward misdirects. Example: `--unset build` while a tier sets `shared_build: true` fails with "true, but the hub declares no build step".
  - Evidence: configcli.go `newLoadFailure` body; `setChecked` and `setHubWide` now apply any `layerWrite`; gateslot config.go shared_build check.
  - Fix: make the refusal text depend on the write kind; for an unset, name keeping the key or fixing the key that depends on it.
- **F3 · NIT · design · `internal/gateslot/reinterpret.go:50`, `if !matches && copies.Scope != ""`**
  - Summary: a copies entry with a non-empty scope that `decodeGates` refuses (the retired `coverage`, or any value but card, diff and all) stops every gate load just as a scope-less one does. It is left and flagged, though the doc comment's stated reason for removing ("it stops the whole gate config loading") applies to it too.
  - Evidence: config.go:562-571 (`decodeGates` scope switch); reinterpret.go:50.
- **F4 · NIT · design · `internal/yamlengine/layers.go`, `RemoveKeys` loop**
  - Summary: a key named twice (`--unset a --unset a`) is removed by the first pass. The second reports "no key a in the file" and refuses the whole write, misreporting a key the file held.
  - Evidence: layers.go `RemoveKeys` loop, where `removeKey` mutates the root in place.
  - Fix: de-duplicate the keys first.

**Tool calls**
1. grep of `SparseBaseHeader` in configengine and yamlengine, and config.go 330-410: `decodeConfig`'s check order was not in context.
2. configengine layers.go 145-175 and the comment and header handling greps: needed to see whether the header survives a sparse write.
3. yamlengine layers.go 425-447: `Prune`'s emptied-file handling, to compare with `RemoveKeys`.

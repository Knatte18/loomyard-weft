```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I traced the callers, tests, docs and rules it forces.

**Per change, the set found and whether the diff covers it**

1. **`gateslot.LoadRefuseRaw` replaces `LoadConfig` in `claudeengine` and `codexengine` `Prepare`.**
   - Callers: `claudeengine.go:163` and `codexengine.go:171`. These are the only callers; every other `LoadConfig` caller is a gate path and keeps the strict load. Covered.
   - Docs: `gateslot/doc.go`, `claudeengine/doc.go`, `Prepare`'s doc in both engines, and the two test docs. Covered.
   - `docs/overview.md:690,723` describe the refuse-raw row and do not mention a load refusal. Nothing stale there.
2. **`RetireHardenLiveCopies` now also removes a scope-less entry, and its flagged text changed.**
   - Registration at `internal/configreg/configreg.go:150` takes ID and func only, so it is unaffected.
   - The test at `reinterpret_test.go` is updated.
   - No stencil, spec or doc quotes the old flagged text or "delete the gates.harden-live-copies entry". Covered.
3. **`dispatch`, `resolvedRefusal` and `runConfig` gain `unsetFlags`, and `setModule`/`setHubWide`/`setLocal`/`setMachine`/`setChecked` take a `layerWrite`.**
   - Every `dispatch` call site in the tests is updated: `configcli_test.go`, `configcli_integration_test.go` and `reconcile_integration_test.go`.
   - The `hubwide.go` import was removed. Covered.
   - `newLoadFailure` (`configcli.go:310`) is now reached by `--unset` but still words its refusal for `--set`. Not covered (F3).
4. **New `configengine.Unset` and `yamlengine.RemoveKeys`, which make `--unset` remove a key the template lacks.**
   - Covered: `configengine/doc.go`, `docs/shared-libs/configengine.md`, `yamlengine/doc.go`, `configcli/doc.go`, the Long help, the reconcile Long and `docs/overview.md`.
   - `docs/shared-libs/yamlengine.md` lacks `RemoveKeys`, but it already lacks `Prune`, `Strip` and `SetSparse` too, so this is no finding.
   - Not covered: `pattern/PATTERN-config-layers.md:7,73` (F1) and `internal/hubreconcile/doc.go:22` (F2).
5. **Help text: "Removing an entry stays an editor edit" was removed and the reconcile sentence reworded.** `TestConfigLong` was updated with `--unset`. Covered.
6. **Commit messages and departures.** No commit in the range carries a `Pattern-Departure:` or `Loosened-Check:` trailer. The `dd4307a5` claim that the way forward names the verb holds at `reinterpret.go:52`.

**Findings**

- **F1 — BLOCKING, design.** `pattern/PATTERN-config-layers.md:7` and `:73`; the code is at `internal/configengine/set.go` (`Unset`) and `internal/configcli/configcli.go` (`--unset`).
  - The rule says "only `lym config reconcile` prunes unknown base-layer keys" and "Only the operator's `lym config reconcile` removes a key the running template lacks, from a hub base or worktree file."
  - `--unset` now removes exactly such a key, a retired key included, from a hub base or worktree file. The new doc text says so at `configengine/doc.go:72`, `configcli/doc.go:70` and `configcli.go:714`.
  - The diff neither updates the PATTERN entry nor carries a `Pattern-Departure:` trailer.
  - Evidence: `pattern/PATTERN-config-layers.md:7` and `:73`, read in this lens's grep.
- **F2 — MEDIUM, consistency.** `internal/hubreconcile/doc.go:22` still says "`lym config reconcile --apply` is the one way to remove such a key". `--unset` makes that false. The diff updated the matching sentences in configcli and configengine but left this one.
  - Evidence: `internal/hubreconcile/doc.go:22`.
- **F3 — LOW, consistency.** `internal/configcli/configcli.go:310`: `newLoadFailure`'s refusal, "the %s config would not load with this value: …; way forward: --set the key to a value it takes", is now also what a `--unset` that newly breaks the load returns, through `setChecked` and `setHubWide`. It names no value and gives the wrong way forward for a removal.
  - Evidence: `configcli.go:310`, and `setChecked` and `setHubWide` in the diff.

**Tool calls**
1. Grep for `harden-live-copies` and the key-removal rule wording across the code, docs, PATTERN and pattern/: needed the full set of readers of the changed messages and rules, which the manifest held only partly.
2. Grep the docs, contracts and plugins for `refuse_raw`/`LoadConfig`, plus users of the reinterpretation and docs of the non-interactive path: needed the docs and stencils that describe the launch's gate load.
3. Grep the headings of `docs/shared-libs/yamlengine.md` and `configengine.md`: needed to know whether the lib docs list each exported function.
4. Grep `configreg.go` for the reinterpretation registration and the repo for `newLoadFailure`'s message: needed the one caller of the changed rewrite and the readers of the message `--unset` now reaches.

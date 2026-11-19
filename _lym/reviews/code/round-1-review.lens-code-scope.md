```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I map each change to its decision and each decision to its changes.

**Per change → decision**

- `gatecli` `--copies`, `copies.go`, `gateslot/startedlines.go`: D4.
- `gateslot` `HubSetsGate`, template and doc gate keys, `goReadGates`: D4, and D2 for Preflight's hub-key check.
- `hardenshed/config.go`, `template.yaml`, `configreg` module, `ConfigSpec` guard keys, `darnengine.NormalizeTrees`: D9. The `NormalizeTrees` export is forced by D9's tree-scope reuse.
- `shedbuild` segmented-build contract, `LoomBuildEngines`: D10.
- `hardenshed/campaign.go` (`CheckCampaign`, `Rotate`), `preflightshed` Harden check, `Env.PreflightHarden`: D2 and D3.
- `hardenshed/live.go`, `loomcli` `hardenLiveRunner`: D4.
- `hardenshed/sandbox.go`, `fabricengine.SeedLocalBares`, `fabriccli.CloneAndWireFromLocalBares`, `lymdirs` constants, `hardencli`: D5 and D11.
- `hardenshed/reap.go`, `reedengine.StopServer`, `runproc` doc, PATTERN change: D5.
- `hardenshed/round.go`, `archive.go`, `campaign_rows.go`: D6 and D7.
- `shedadapters` `RoundFacts`, `LiveRed`, `OnHalt`, `ReviewRound`, `FixedRound`: D6, with D5's halt reap.
- `shedrecipe` `HardenFix`, `harden_round`, commit seam `harden`: D3, D7 and D10.
- `harden-rubric-review` stencil, `stencils.go`, `.gitattributes`: D3, D6 and D12.
- `hardener-recipe.yaml`, `loomrecipe`, `shedcli`, `shedrun`, `battencli`, `reportengine`, `boardcli` and `boardengine` recipe wording, `loomshed` names and interrupt policies: D1.
- `loomcli` hardenwiring, `treeScopeSettings`, `recipeTrees`, `perchHardCap`, `reviewBudget`: D1, D3 and D9.
- `landingshed` `Reap`: D5 (Finalize).
- `pairteardown` `clearSandbox`: D5.
- `agentconfig` `DropRetiredRoles` and `unusableWithWayForward`, `configreg` reinterpretation, treadle deletion, `stencils.go` treadle removal, `shuttleparity` and `cmd/lym` test edits: D8.
- Treadle comment cleanups in `mergeresolve`, `shedengine`, `verifyrun` and `webstercli`: forced by D8.
- `crucible/` deletion, `fabricvocab` doc roots, `codestats`, docs, `PATTERN-fabric-boundary`: D12.
- `hardenshed/doc.go` Method section: D12.
- `cmd/lym/main.go` and its tests: D11.
- PATTERN background lines for `cli-cobra`, `config-layers`, `gate-slot`, `test-speed`, `cwd-resolution`: forced by new packages and the treadle removal.

**Per decision → changes**

- D1–D12 each have at least one change.
- The parent's added entry (hub gate keys as operator writes) is honoured: no card writes the hub values.

**Findings**

1. LOW, class `scope`, `internal/boardcli/cli.go` (hunk at "notify_on_land"): the help text for `notify_on_land` changes from "tell the orch when the run lands, at once if it has" to "tell the orch when the run lands". No decision asks for it, and the hardener "recipe" line in the same hunk does not force it. Fix: revert that wording line.

2. LOW, class `scope`, `internal/configcli/configcli.go` (`buildConfigLong`): "edits that module directly" becomes "edits that module". It traces to no decision. Fix: revert.

3. LOW, class `scope`, `internal/gatecli/cli.go` (`--upto` and `Each tier` help lines): the text is reworded beyond the `--copies` line. The edits drop "through the platform shell", drop the "Inside a slot an enclosing gate run holds it acquires no other" sentence, and reword the `--upto` line. D4 only asks for the `--copies` flag. Dropping the inherited-slot sentence also weakens documented behaviour that still holds. Fix: restore the original lines and keep only the added `--copies` line.

4. LOW, class `scope`, `docs/overview.md`: the "internal/treadleengine" tree line is replaced by an `internal/hardenshed` line. That is licensed by D12 and card 20, so it is not a finding. No other unlicensed hunks found.

Only findings 1–3 are real (LOW); finding 4 is noted and rejected. Dropped: D8 and D12 comment edits in `mergeresolve`, `shedengine`, `verifyrun` and `webstercli`, forced by the treadle removal.

Tool calls: none. The manifest already held everything this lens needed.

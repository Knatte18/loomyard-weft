---
dispositions:
  - id: N1
    disposition: fixed
  - id: N2
    disposition: fixed-differently
    reason: "The proposed refusal fires only when the call names no package target and no filter; the self-check showed that leaves `--tier 2 ./...` and `--tier 2 -- -run X` open, and both start tests of every package in scope. The refusal now fires unless the call names an explicit target free of `...`, and a filter alone does not open it. Evidence: internal/gatecli/run.go namesPackage, internal/gatecli/cli_test.go wildcard_target and filter_after_dashes_alone rows."
---

# Round 2 fixer-report

## What changed

- **N1** (commit `shellguard: fix N1`).
  - `shuttleengine.Spec` gains `AllowRawTmux`, documented as the exemption from the tmux and script deny (`internal/shuttleengine/spec.go`).
  - The sampler's spec sets it (`internal/samplerengine/spec.go`), and its test pins it.
  - `SwitchAutonomous` is replaced by `SwitchDenyRawTmux` in `internal/shellguard/rows.go`: installed and announced unless the spec sets the exemption.
  - `claudeengine` maps it from a new `denyInputs.allowRawTmux`, and `codexengine` from a new `guardInputs.allowRawTmux`, both fed from `spec.AllowRawTmux` and never from `Interactive`.
    `buildSettings` and `buildDenyNotice` take the new bool, and every caller passes it.
  - `steerTmuxScriptDeny` drops "in an autonomous run" and says "in this session".
    The row comment, the switch doc, `internal/shellguard/doc.go` and `claudeengine/doc.go` say the row reaches every run session but the sampler.
  - `claudeengine/testdata/settings-golden.json` is regenerated with a throwaway test, since deleted.
  - The engine tests expect the row on an interactive spec and its absence on a spec that sets the exemption:
    the Claude and Codex manifest tests, `TestBuildSettings_NoAgentHookAndForkGuard` (new `allowRawTmux` field and an exempt interactive row, plus the notice sentence),
    `TestBuildSettings_DenyToggleMatrix`, `TestBuildDenyNotice_SentenceContent`, the Codex `installedGuards` table and `TestPrepare_InstallsGuardsFromItsInputs`.
    The Codex tests that carried `autonomous: true` are back to the zero `guardInputs`.
  - `_lym/landing/summary.md` carries the new switch, the design decision and the tests.
    I edited it and committed nothing under `_lym`.
- **N2** (commits `gatecli: fix N2` and `gatecli: fix N2 self-check`).
  - `gateslot.TmuxTier` names tier 2 beside `BilledTier`.
  - `testRequest.runAgent` is set in the verb's flag parsing from `agentname.StrandNameEnv`.
  - `flagShapeRefusal` refuses a run agent's call whose `--tier` or reaching `--upto` lists the tmux tier and names no package.
    It runs before hub resolution and before the front, so a detached runner, which has no strand name, never bypasses it.
    The refusal names `lym gate test --tier 2 <the package that holds the test> -- <a filter selecting one test>`.
  - `internal/gatecli/doc.go` documents it, and `TestFlagShapeRefusal_RunAgentTmuxTier` asserts the full refusal text and the cases that pass.
  - `_lym/landing/summary.md` replaces the no-gate-side-check decision with the check.
- Gate repair: two comments the round added are now one sentence per line, in `internal/gatecli/cli_test.go` and `internal/shuttleengine/codexengine/guards_test.go`.
- Gate: `lym gate test --tier 0 --tier 1` over `gatecli`, `gateslot`, `samplerengine`, `shellguard`, `shuttleengine`, `claudeengine` and `codexengine` passed, and `gatecli` passed again after the self-check change.

## Self-check

Fork lenses ran once; their reports are saved beside this file.

- fix-code-correctness, LOW, `--upto` refusal can fire for a tier a hub marks manual or leaves undeclared: disputed, `every fix worse`.
- fix-consistency: no findings.
- fix-proposal, MEDIUM, `./...` and a filter alone pass the refusal: fixed differently, as N2's own change above.
- fix-proposal, LOW, `TmuxTier = 2` is hardcoded: disputed, `premise false`.
- fix-proposal, LOW, `--gate`, `card-verify` and `red-first` calls are not refused: disputed, `every fix worse`.
- fix-proposal, LOW, the orch stencil tells the orch to run `tmux capture-pane`: disputed, `premise false`.
- fix-proposal, no finding on the N1 mechanism.
- fix-affected-set: no findings.

## Disputed

- **fix-code-correctness LOW, `--upto` false positive** — ground `every fix worse`.
  The refusal cannot read the hub's tier declarations, since it runs before the config loads.
  Moving it after the config load lets a detached runner, whose environment has no strand name, run the call unrefused: `internal/gatecli/detachfront.go:258` caps a call only when `flagShapeRefusal` is empty, and `internal/gatecli/run.go:43` is where it refuses.
  The refusal's way forward still opens the call, by naming a package.
- **fix-proposal LOW, hardcoded `TmuxTier`** — ground `premise false`.
  `pattern/PATTERN-test-speed.md:14` fixes `tier2` as the tmux tier of every hub, as `internal/gateslot/startedlines.go:15` fixes `BilledTier`.
- **fix-proposal LOW, `--gate`, `card-verify` and `red-first` unrefused** — ground `every fix worse`.
  Those calls take their tiers and scope from `gate.yaml`, and `pattern/PATTERN-test-speed.md:14` has keys list the tmux tier.
  Refusing them would break the card gates that list it.
  The decision asks for a refusal of a run agent's tmux-tier run through `--tier`, and `internal/gatecli/doc.go` states the exemption.
- **fix-proposal LOW, orch stencil** — ground `premise false`.
  The orch runs as an opened session: `internal/shuttleengine/open.go:94` calls `OpenLaunch`, and `internal/shuttleengine/claudeengine/settings.go:233` writes its settings with creel hooks and no shell rows.
  The row therefore never reaches it.

## Deferred

Nothing.

## Rulings

No parent ruling reached me this round beyond the added entries F4 and F5, which the review raises as N1 and N2 and which are fixed above.

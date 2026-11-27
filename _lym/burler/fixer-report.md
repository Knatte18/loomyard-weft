---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed test runs root's pre-run, but the pre-run installs the reader only when testing.Testing() is false (cmd/lym/main.go, the `if !testing.Testing()` block around InstallPassageReader), so no test can get the reader that way. The closure is now the named function readStandingPassages, which the pre-run installs and the new test installs and calls."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
---

# Fixer report

## Changes

- F1: `burlerSeatTables` also builds the unassisted solo round (no role prefix) and the unassisted fanned round (`unassisted-fanned`).
- F2: `heddleengine.ClaudeLocalFile` is the one declaration of the file name; the target row, `InstructionFiles` and the manifest use it.
- F3: one constant per engine name in `heddleengine/target.go`, used by the file and prompt rows.
- F4: `LookupSeat` resolves a burler role under any prefix, or none, to the catalog seat of that role.
  The assistant's role resolves to the assisted solo round's assistant, since no unassisted round has one.
  `fixRolePrefix` is exported as `FixRolePrefix` for this; the verb's help and the package doc state the rule.
- F5: an empty instruction file in the config directory or above the repository is not listed.
- F6: the parity fixture has a hub `gate.yaml` with `refuse_raw`, an anchor below the root and a root `CLAUDE.md` and `AGENTS.md`;
  `TestSeats_CodexManifestHoldsClaudes` asserts both items stay in the Claude manifest.
- F7: `readStandingPassages` and `cmd/lym/standingpassages_test.go`.
- F8: one negative test per engine over candidates the launch does not hold.
- F9, F12: new assertions in `TestCompare`.
- F10: `TestWorktreeManifestReport_RunsNoPluginHook`, and `TestManifestCmd` asserts no `verdict` key.
- F11: each user-setup warning is logged.
- F13, F14, F15, F16: new rows and assertions in the manifest tests.

Verification: `lym gate test` over the changed packages, tier0, 0 failed.

## Disputed

(none)

## Deferred

(none)

## Rulings

(none)

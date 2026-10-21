# Fixer report — discussion, round 1

Every finding's premise was checked against the code (`internal/configengine/config.go` `load`, `layers.go` `Provenance`, `internal/gateslot/config.go`, `internal/configcli/hubwide.go`, `configcli.go`, `configengine/edit.go`, `configsync/movekeys.go`) and holds.

## Changes

- F1 — fixed as proposed: D1 states the rule reads the told `Dirs`, so an anchor without `_lyx/` still stacks the machine layer; criterion added. Plan-writer note added that the rule is judged before `load` and `Provenance` clear `Anchor`.
- F2 — fixed as proposed, then sharpened by the self-check: D3 names both files only when this machine's file is in the stack, and the hub file alone for a `Board`-only load or with no machine file present; the Board-only criterion now requires the refusal to name neither the machine file nor `--layer machine`; Scope's message bullet updated to match.
- F3 — fixed as proposed: D5 states a retired `machine` key is admitted and dropped silently and any other retired key fails with the D3 message; criterion for `cli_wait_sec` added; D5's rationale and support log Q7 updated to match.
- F4 — fixed as proposed: D3's ways forward are each named from the prime, including removal; a Bound paragraph added; the `lyx gate test` criterion requires "from the prime".
- F5 — fixed differently. The proposed D5 sentence ("for a hub-wide module with no `machine` key any present unit is refused") was made, then replaced after the self-check: it left a stray `fabric.machine.yaml` failing every fabric load with no way forward, contradicting D5's rationale that limits a bad machine file to gate loads. Instead D1 stacks the machine layer only when `MachineKeys(spec)` is non-empty, so D5's "and nothing else" holds; Scope, D4 (refusal of `--layer machine` on a hub-wide module with no `machine` key, naming `lyx config <module>` from the prime), Constraints and two criteria updated; support log Q9 updated.
- F6 — fixed as proposed: D4 separates `--set` (refused before the write) from an editor save (checked after, put back or removed); criterion and Constraints entry added.
- F7 — fixed as proposed: D2's Bound states a direct machine-file write never reaches Bolt or any record, and that no upper bound applies.
- F8 — fixed as proposed: D4 drops the menu from the `--layer machine` list.
- F9 — fixed as proposed: D1's rationale names the `<module>.yaml` and `<module>.local.yaml` names instead of the wrong staging path.
- F10 — fixed as proposed, with "only reconcile moves keys in config files", since `configreg/runsettings.go` also applies `KeyMoves`, to run settings only.

## Self-check

- fix-claims, LOW, D3 Bound overclaims that hub-file checks keep the hub file valid — fixed as proposed: "unless the hub file was written directly".
- fix-consistency, MEDIUM, stray non-gate machine file refusal has no way forward or Constraints entry — fixed differently (see F5): the file is never read.
- fix-consistency, LOW, editor-save refusal has no criterion or Constraints entry — fixed as proposed.
- fix-consistency, LOW, Scope's message bullet left without the Board-only exception — fixed as proposed.
- fix-consistency, LOW, D5 rationale "the D3 message fixes it" stale for a retired `machine` key — fixed as proposed.
- fix-way-forward, MEDIUM, same as fix-consistency's MEDIUM — merged, fixed differently (see F5).
- fix-way-forward, LOW, no-`_lyx/` criterion depends on judging the rule before the anchor is cleared — fixed as proposed: plan-writer note added.
- fix-proposal, MEDIUM, F5's sentence contradicts D5's rationale — merged with the MEDIUM above, fixed with the fork's proposed alternative.
- fix-proposal, MEDIUM, removal way forward steers pair sessions around D4's guard — fixed as proposed: both machine-file ways forward are named as prime steps, and the Bound says so.
- fix-proposal, LOW, same as fix-claims' LOW — merged, fixed.
- fix-proposal, LOW, "a stack that holds the machine layer" is ambiguous when the file is absent — fixed as proposed: "a stack in which this machine's file is present".

## Disputed

None.

## Deferred

None.

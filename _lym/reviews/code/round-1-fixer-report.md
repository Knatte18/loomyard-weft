---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed `refuse_raw: 3` row was not added. A scalar where the template holds a list fails the layer merge in configengine.Load (layers.go:267, shape mismatch), so the row returns at the merge failure and never reaches the fallback's decode checks; it duplicates the unparsable-file row. Only the `go  test` row, which reaches the fallback, was added to both engine tables."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The proposed text 'keep the key, or --set the keys it leaves invalid' misdirects a --set that itself writes an invalid value, as the fix-proposal fork showed. The text instead reads 'way forward: --set the keys it leaves invalid to values the loader takes, or leave the key as it is', which fits both writes without passing wording in from the layerWrite."
  - id: F7
    disposition: fixed-differently
    reason: "The proposed scope check (diff or all) misses an entry decodeGates refuses for lints, reports or a fixer on a non-card key. The removal is decided by calling decodeGates on the entry, so the rewrite removes exactly what stops the gate load; the proposed rows for `{tiers: [oops], scope: diff}` and `{tiers: [1], scope: coverage}` were added."
  - id: F8
    disposition: fixed-differently
    reason: "The worktree row now asserts the key left the file, and a row with one present and one missing key wants the file unchanged. The proposed absent-file row on the hub-wide table was not added: that table reads the hub file after every row and fails the test when it is absent. TestDispatchSet gets a row for a key the worktree file does not hold instead; Unset's absent-file refusal itself has no dispatch row."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The billed tier clause is derived with fmt.Sprintf from gateslot.BilledTier instead of the literal 'one tier below tier3', so a change to the constant does not fail the test for the wrong reason; the unset-step assertion is as proposed."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Round 1 fixer-report

F4 is `decision`-class and held by the parent; it is neither fixed nor reported here.

## What changed

- **F1** `pattern/PATTERN-config-layers.md`: the Layers rule and "Keys the template lacks" now say the operator's `lym config reconcile` and `lym config <module> --unset <key>` are the two removals of an unknown key; the ban on every automatic removal and its reason stay.
  The one-line entry in `PATTERN.md` never mentioned reconcile, so it still matches.
- **F2** `internal/hubreconcile/doc.go`: names both operator routes.
- **F3** `codexengine/prepare_test.go` and `claudeengine/settings_test.go`: a row with an invalid `refuse_raw` list beside the scope-less `harden-live-copies` entry wants the refusal, before any artifact in the Claude table.
- **F5** `internal/gateslot/config.go`: `LoadRefuseRaw` returns the `refuse_raw` decode error with the gate config way forward; the doc comment says so.
- **F6** `internal/configcli/configcli.go`: `newLoadFailure` words its refusal for both writes; the two tests that asserted the old text use the new one.
- **F7, F10** `internal/gateslot/reinterpret.go` and its test: the rewrite removes an entry the gate load refuses, whatever its tiers; `decodeGateKey` returns whether the entry decoded; the flagged line says `did not decode` for an undecodable entry.
  The test asserts the billed tier clause on every flagged line and the unset step on the scoped ones.
- **F8, F9, F14** `internal/configcli/configcli_test.go`, `internal/yamlengine/layers.go`: the new rows for the worktree file content, the all-or-nothing list, the missing-key refusal, `module required with --unset` and `--resolved` beside `--unset`.
  `RemoveKeys` skips a key it already removed, covered by a hub-wide row naming `labels.old` twice.
- **F11, F12** `configcli_test.go`: keep mark on `TestDispatchSetProfiles`; `--unset` dropped from `TestConfigLong`.
- **F13** `_lym/landing/summary.md`: the blast radius names shuttle launches, `lym config` writes on every layer and the gate reinterpretation.
  The file sits under `_lym`, so it is edited and not committed.

Gate repair: the `TestRetireHardenLiveCopies` doc comment in `internal/gateslot/reinterpret_test.go` wrapped mid-sentence and ran to 120 characters.
It now holds one sentence per line, each under the limit.

Gate: `lym gate test` over `gateslot`, `configcli`, `yamlengine`, both shuttle engines and `hubreconcile` passed at tier0, and again over the four packages the self-check changes touched.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, `newLoadFailure` text misdirects a `--set`: fixed differently, as F6 records.
- fix-proposal, LOW, `stopsLoad` misses lints, reports and fixer: fixed, through `decodeGates`.
- fix-proposal, LOW, `tier3` literal pinned in the test: fixed, derived from `gateslot.BilledTier`.
- fix-proposal, LOW, `refuse_raw: 3` rows may never reach the fallback: fixed, rows dropped.

## Disputed

None.

## Deferred

None.

## Rulings

None.

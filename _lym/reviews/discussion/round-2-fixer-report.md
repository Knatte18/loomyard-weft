---
dispositions:
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed-differently
    reason: "The takeover half is as proposed. For the code-tip settle, the proposal has fabric merge the seam's encoded bytes over the current file through internal/state. That merge cannot keep shed's unknown keys, because internal/state keeps unknown keys by the written Go type (internal/state/keep.go:34) and treats json.RawMessage as opaque (keep.go:58). So shed's adapter runs the rewind write through state.WriteJSON with shed's status type, as codetipsettle.go:90 does today, and fabric then commits."
---

# Round 2 fixer-report: decision record

## Changes

- **F20, fixed** (`_lym/discussion/decision-record.md`):
  - D13 adds one installer of the required seams in `internal/fabriccli`: shed's adapter for run-records paths, run locks and codec, and the stencil ordering judge.
    `cmd/lym/main.go` calls it.
    `internal/hubforge`, which already imports `internal/fabriccli` (`hub.go:18`), calls it once per test binary and never restores it.
  - Tests outside fabric that read the hub workspace file install the IDE module's builder themselves, as `internal/ideengine/spawn_hub_integration_test.go:180` does today.
  - D13's fabriccli clause now says the cut extends fabriccli's imports only by the installer's adapter imports.
  - D5's sentence on test binaries names the installer.
  - A new criterion says a hubforge-built pair outside fabric is removed unrefused.
  - Auto-mode assumptions gain the installer.
- **F21, fixed differently** (`_lym/discussion/decision-record.md`):
  - D13's Writes sub-bullet now sends both writes through `state.WriteJSON` under the status lock.
    The takeover write is a one-key patch with fabric's own takeover type.
    The code-tip rewind is written by shed's adapter with shed's status type.
    The frontmatter reason says why the rewind does not merge bytes in fabric.
  - The same-bytes criterion now starts from a status file holding a key shed's type does not declare.
  - Auto-mode assumptions gain the locked-merge write.

## Self-check

My change set got four lens forks.
I fixed every fork finding as proposed in one further round of edits to `_lym/discussion/decision-record.md`:

- fix-claims: no findings.
- fix-consistency, MEDIUM, Scope did not admit the installer, hubforge's call or the builder installs: fixed. Scope In gains a bullet, and the Scope Out bullet on other packages' tests now admits D13's builder installs.
- fix-consistency, LOW, D9 placed no part for the installer: fixed. D9 says the first part that makes a seam required adds the installer and hubforge's call, and the builder part adds the outside tests' installs.
- fix-consistency, LOW, the shed-records lead-in still said writes cross as codec bytes: fixed. It now says reads cross as bytes and writes go through `internal/state`.
- fix-consistency, LOW, the builder's nil rationale cited D12 against the real builder: fixed. The rationale now says no fabric rule reads the file, and the installer paragraph says why the builder stays out of it.
- fix-consistency, LOW, and fix-proposal, MEDIUM, merged: hubforge's real seams have fabric's tests call shed's codec and the stencil judge, which D12's hubforge exemption did not cover. Fixed: D12's exemption now covers the real required seams the installer puts in, taken as fixture with only fabric's own state asserted. D13 says a fabric test that reaches a codec or judge call on a hubforge hub asserts no verdict of it.
  This widens how D12's operator decision is read. The parent may want to confirm it with the operator.
- fix-proposal, MEDIUM, and fix-way-forward, both MEDIUM findings, merged: the restore races parallel tests, and the refusal test has no path to an unset seam. Fixed: hubforge installs once per binary and never restores. The installer package offers a clear that returns a restore. The refusal test builds its hub, clears the seams, restores them before returning and does not call `t.Parallel`, since Go runs parallel tests only after the sequential ones. D5 now names the clear.
- fix-proposal, LOW, fabric's takeover type can drift from shed's: fixed. The same-bytes criterion adds a test in shed's package that fails when the two types declare different JSON keys.

## Disputed

None.

## Deferred

None.

## Rulings

None.

# Round 2 fixer report — discussion

## Changes

- F1 (fixed as proposed): D2.8 now has the gate closure record each `fix:` commit's SHA and subject and each `Pattern-Departure:` trailer into its `VerifyGateNotes` at each evaluation, and Run writes them after the wait into `summary.md`'s `## Verify gate fixes` section, the way `VerifyGateNotes.Apply` writes the flaky identities.
  The Master's step-4 section names only the findings from the findings file.
  D2.4's clause on the gate result the Master re-arrives with is dropped.
  The Acceptance criterion now states that after a fix the gate passes on, `summary.md`'s section names each fix commit and trailer, written by Go after the wait, and a fix with no trailer names no departure.
  Also updated for consistency: the D2.8 entry in `## Auto-mode assumptions` and Q5 in the support log's question ledger.
  Premise checked: `webster-template-master.md` step 4 writes the summary before the re-arrival, and `internal/websterengine/doc.go` has the gate record the pre-fix head and walk the fixer's commits only at the next arrival.
- F2 (fixed as proposed): D5.4's way forward now reads that the operator fixes and commits the named section, the orch never editing the record and escalating the refusal to the operator; the `validate-discussion` check and re-step stay.
  The matching Acceptance criterion names the operator's record fix, and Q13 names the operator.
  Premise checked: `contracts/stencils/orch/orch-template-role.md:106` forbids the orch to edit or commit the decision record.
- F3 (fixed as proposed): D5.7 states the landing check reads the declared entries from the record's `## PATTERN changes` as rendered into the Webster-Review directive by D5.4, never from the plan's section.
  D5.3 states a bullet appended after the plan gate reaches the plan's section only at the next plan gate, where `pattern-changes-mismatch` asks for it.
  A new Acceptance criterion states such a bullet is rendered into every later directive and checked by D5.7.
- Support log: a `Round 2` line under `## Review rounds`.

`lyx loom validate-discussion` passes.

## Disputed

None.

## Deferred

None.

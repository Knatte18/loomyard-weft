```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each hunk of the diff to the goal points of the board entry.

**Decisions.** These are the board entry's `## Goal` points. The record holds no `Added after Discussion` entries.

- **G1.** The card gate's listed run covers the card's own packages only, and dependents run at the batch gate, for Webster and Weaver alike.
- **G2.** The card gate's fixer re-runs only the check that failed, and leaves the whole card gate to the call that judges the fix.
- **G3.** The guard step runs only in the batch gate, if the measured trade still holds, and the change description says why.
- **G4.** The plan-level verify keeps running tier1 over everything.

**Per change**

- **C1. `internal/gatecli/scope.go` `computeScope` `ownOnly` branch.** It uses own packages, sets `guardStep: !ownOnly` and prints the stderr line. Traces to G1 and G3. Weaver reaches it through `weaverengine/strand.go` `batchGateCalls`, which uses the same `card` key.
- **C2. `internal/gatecli/scopes_integration_test.go` rows.** Forced by C1.
- **C3. Docs of the new split.** The `doc.go` paragraphs, the `cli.go` `--card` help, `contracts/specs/loom-plan-spec.md`, the `gateslot/template.yaml` comment, `pattern/PATTERN-test-speed.md` and the `contracts/stencils/govalues_test.go` file comment. Forced by C1, as docs it made stale.
- **C4. `webster-body-implementer.md`.** It moves the guard-test fix rule to the batch gate section and fixes the sanctioned-deviation pointer. Forced by G3.
- **C5. `weaver-template-batch.md`, the line on what the batch gate adds.** Forced by G1 and G3.
- **C6. `contracts/stencils/gatefix/gatefix-body-card.md`, the re-run rule and the never-run-the-whole-call rule.** Traces to G2.
- **C7a. The `cardGateKey` → `CardGateKey` export in `gateslot/tiers.go`, with its uses in `config.go` and in `goReadGates`.** Forced by C1, which reads `gateslot.CardGateKey` from `gatecli`.
- **C7b. Other gate-key literals replaced.**
  - `gateslot/priority.go`: the duplicate `batchGateKey` is deleted and both key checks now use the exported constants.
  - `weaverengine/strand.go`: `gateKeyCard` is deleted and its uses are replaced.
  - `runplan/cardgate.go` and `webstercli/beginbatch.go`: the `"card"` literals are replaced.

  No goal point asks for this. C1 does not need these sites changed, and no `Pattern-Departure:` trailer names it. `PATTERN-code-conventions` ("a name or path that can go stale is one named constant") supports it, but the duplicates were there before this change. This is unrequested scope, but small.

**Per decision**

- **G1:** C1, with C2, C3 and C5 forced.
- **G2:** C6.
- **G3:** C1 (`guardStep: !ownOnly`) and C4. The why is in `_lym/landing/summary.md` lines 35–44, which is G3's "says why in its change description". The figures themselves are for the handler and other lenses to check.
- **G4:** no change needed, and none made. Nothing in the diff touches the plan-level `webster` key.

**Findings**

1. **LOW, class `scope`.** Location: `internal/gateslot/priority.go:31-46`, `internal/weaverengine/strand.go:21-29`/`:107-139`, `internal/runplan/cardgate.go:28`, `internal/webstercli/beginbatch.go:64,76`.
   - **Summary:** the gate-key cleanup outside `gatecli`/`gateslot` traces to no goal point and is not forced by C1. It deletes the duplicate `batchGateKey` and Weaver's `gateKeyCard`, and replaces the `"card"` literals in `runplan` and `webstercli`.
   - **Evidence:** diff pieces `007-internal-gateslot.diff` lines 27–43, `009-internal-weaverengine.diff` lines 9–29, `008-internal-runplan.diff` line 10 and `010-internal-webstercli.diff` lines 9 and 19, against decision-record.md lines 15–23.
   - **Mitigation:** the edits change no behaviour, and `PATTERN-code-conventions` (one named constant per name that can go stale) supports them. A fix is either to keep them and name them as rule-driven cleanup in the change description, or to drop them to a separate task.

No tool calls made.

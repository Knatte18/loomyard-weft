```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, and checked the acceptance criteria and constraints against those decisions.

**Pairs checked**
- D3 × D14 (in-flight `scope.json` holding `partial`, after `partial` is removed): broken
- D3 × D15 (`drift.scope: partial` in a hub base now means `full`; the reinterpretation is silent): broken
- D3 × D4 (scope recorded once; later rows read only the record): held
- D3 × D8 / D12 / D13 (`off` and deferred pass; the verdict states the source): held
- D2 × D6 × D8 (seal check versus the `command`-mode pass of the gate entry and `lym drift check`): broken
- D2 × D4 ("passing check" skip rule under `command` mode): broken (undefined)
- D2 × D11 (amendments under `command` mode, where the judge decides containment): broken
- D9 × D11 (amendment `why` reaches the judge, but D11's bound says "reaches no agent"): broken
- D4 × D6 (in-flight run past Drift-Expect checked against an empty expectation): held for the two decisions together; how code tells "past" is the handler's focus
- D5 × D6 (exhausted `expectation` entry Stuck, then re-entry spawns again or seals a valid file): held
- D6 × D4 (Warp-SHA equals baseline head; no warp commit between Drift-Baseline and Drift-Expect in either recipe): held
- D6 × D11 `--restore` (writes the sealed copy back; seal commit untouched): held
- D6 × D12 (deferred capture: no seal, and every check passes): held
- D4 skip rule × D10 / D11 (amendment forces a full pass; only unruled tuples are judged): held
- D7 × D8 (own scratch directory, capture lock, inherited slot): held
- D9 × D4 (judge writes only `judgement.yaml`; Drift-Verify edits no code): held
- D9 × D10 (rulings stored as tuples; `judge_interactive` rulings stored the same way): held
- D10 × D11 (adding entries drops the covered rulings): held
- D12 × D8 / D4 (deferral passes every row and the gate entry): held
- D13 × D4 / D11 / D12 (verdict rendered from `scope.json` and `judged.json`): held
- D15 × D2 (`diff` hub-only; reinterpretation for `compare`): held
- D16 × D6 / D8 (writer stencil says the sealed file is not to be edited): held
- AC D6 × D6 ("hand-edited record"): broken (stale)
- AC D8 × D7 (no tracked record; scratch directory under `.lym`): held
- Constraints (sole parsers, tolerant records, durable vs ephemeral, recipe registry, fabric isolation): held against D4–D10

**Findings**

1. **BLOCKING · design · D3 with D15.**
   D3 reads a configured `drift.scope: partial` as `full`, so a hub value's meaning changes.
   D15's `configreg.Reinterpretation` flags only a set `compare` with no `diff` and commands holding `{skip_file}`.
   A hub base that sets `scope: partial` is never flagged or rewritten; the only signal is a per-run Warn.
   This breaks `PATTERN-config-reinterpretation`: a binary that changes a hub config value's meaning must carry a reinterpretation that rewrites or flags it.
   Evidence:
   - decision-record.md:83 and :266-269
   - `internal/driftengine/config.go:192-200`: `partial` is a valid scope today, with `skip` required.
   - `internal/driftengine/dirbase.go:16-17`: the module already has a reinterpretation of this kind.

2. **MEDIUM · design · D3 with D14 (in-flight runs).**
   D3 keeps a recorded `off` or `full`, and D14 removes `partial` from the code.
   Nothing says what Drift-Verify, Drift-Recheck or D13's verdict do with a `scope.json` that already records `partial` and its `skip` list.
   D4's resume text and the Open risks line cover only "past Drift-Expect".
   Today's readers branch on `ScopePartial`, so after the change such a record reaches undefined behaviour.
   Evidence:
   - decision-record.md:84, :249 and :316
   - `internal/driftengine/records.go:20-23,71-74`
   - `internal/driftengine/verdict.go:69-74`
   - `internal/driftengine/verify.go:612`

3. **MEDIUM · consistency · D6 against D8 and D2 under `command` mode.**
   D6 says the drift gate entry and `lym drift check` check the seal first "in either diff mode", and a failed seal check is a finding.
   D8 says the entry "passes … in `command` mode".
   D2 says both pass and report the compare's exit and log path, which implies they run the compare, a step D8 does not list.
   AC D6 ("the seal is checked in `command` mode too") and AC D8 ("passes under … `command` mode") pin opposite results for a broken seal at the gate entry.
   Evidence: decision-record.md:71, :140, :171, :325 and :327

4. **MEDIUM · design · D11 with D2 (amendments under `command` mode).**
   In `command` mode the judge does the containment, reading "the compare's log and the expectation" (D2).
   D11 says containment reads the sealed expectation plus every amendment, but its bound says an amendment "reaches no agent of the run".
   For a `command`-mode Stuck, either the amendment never reaches the judge, so `lym drift amend` is not the way forward D11 names, or the bound is false.
   Evidence: decision-record.md:68, :216 and :219

5. **LOW · consistency · D11's bound against D9.**
   D11 says amended cells are "judged again against the amended `why`", and D9 hands the judge "the covering entry's `why`", which can be an amendment's.
   So an amendment does reach the judge, which contradicts the bound "reaches no agent of the run".
   The bound presumably means no writer or expect agent.
   Evidence: decision-record.md:184, :217 and :219

6. **LOW · consistency · D4 against D2.**
   D4 defines a passing check only in cell terms ("every cell expected and ruled `matches`").
   Under `command` mode there are no cells or stored rulings.
   It is undefined whether a compare that fails but has every case ruled `matches` counts as passing.
   That decides whether Drift-Recheck at the same head is Done at once or judges again, and D2's "each failing compare is judged again" suggests judging again.
   Evidence: decision-record.md:70 and :104-105

7. **LOW · consistency · AC D6 against D6.**
   The criterion lists "an edited file with a hand-edited record" as a seal-check failure.
   D6 keeps no seal fields in any record, so the criterion names a mechanism the decision removed.
   It is left over from the record-based seal the support log says was rejected.
   Evidence: decision-record.md:138 and :325; support-log.md:73

Outside my lens, noted only: D14 removes Drift-Verify's `verify` gate entry, but D4 does not say what happens to the Drift-Verify row's `gate_key: drift-verify` and `gates: [verify]` config, or to the gate key itself (tapestry-recipe.yaml, Drift-Verify row; `internal/gateslot/template.yaml`). That belongs to the existing-paths lens.

Tool calls:
1. grep for `reinterpret` in `internal/driftengine`: the manifest did not show whether the drift module already has a reinterpretation, which finding 1 needed.

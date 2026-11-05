```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the record's decision pairs for one decision undoing, outrunning or contradicting another.

**Pairs checked**
- D6 re-entry × D14 removals: broken (finding 1).
- D3 in-flight `partial` and D13 verdict × D14 and D15 removing `skip`: broken (finding 2).
- D1 identities, D5 seal and D10 stored tuples × D15 hub-only `diff` and `key_patterns`, with nothing fixed per run: broken (finding 3).
- D2 `command` mode × D5 validation and D11 amendments that use D1 identities: broken (finding 4).
- D4 "Done at once only at the last passing head" × D3 scope `off` and D12 deferral: broken (finding 5).
- D8 gate entry × D12 missing baseline: broken (finding 6).
- D4 Done rule × D11 amendments forcing a re-check: held.
- D4 × D10 judging only unruled tuples: held.
- D10 stored `does not match` × D11 amendment dropping covered rulings: held.
- D7 classifying before the judge × D11 `--discard` leaving stored `matches` rulings: held. Containment classifies first, so a cell the discard uncovered stops as unexpected.
- D2 passing check × D4 Done rule (judge rules every case `matches`): held.
- D6 seal check × D8 entry reporting without failing: held.
- D6 failure ways forward × D11 `--restore` and `--discard`: held.
- D6 "past Drift-Expect" by row order × the D4 row order in tapestry and darn: held.
- D6 seal `Warp-SHA` equal to the baseline head × tapestry's Plan/Parts writers, which commit weft only: held.
- D6 × D12 deferral (Drift-Expect Done at once, no seal): held.
- D7 scratch capture and inherited slot × D14 removing the `drift-verify` gate key: held.
- D7 goto route × D4 Drift-Recheck losing `on_stuck`: held.
- D9 judge tree guard × D10 and D11 writes (written only by the row or at a Stuck, never during the judge): held.
- D9 `open_ranges` × D10 stored ranges, when every cell reuses a ruling: held.
- D11 admitted rows × D8 (an exhausted `drift` entry makes the writer Stuck): held.
- D12 × D11 (defer allowed at a Stuck Drift-Expect, amend not): held.
- D13 × D14 keeping the old compare, acceptance and waiver fields read-only: held, apart from `skip` (finding 2).
- D15 retired `skip` in `run_settings` × D3 reading `partial` as `full`: held. `ValidateLocal` admits a retired CategoryNone key and the load drops it (`internal/configengine/layers.go:291-299`).
- D15 `{skip_file}` refusal × D2 `command` mode and D12: held. Each names the edit.
- Acceptance criteria D2, D6, D9, D10 and D11 against their decisions: held.
- AC D4 against D12 and D3 `off`: broken (finding 5).

**Findings**

1. **MEDIUM · consistency · D6 × D14.** D6's re-entry rule uses an agent-session mark that D14 deletes.
   - D6 seals an existing `expected.yaml` without a spawn only when the expect session "ended normally, recorded as today's drift producers record an open agent session (`AgentSessionOpen`)".
   - That mark lives in `runs.json` (`internal/driftengine/records.go:144-151`, `:277-281`), which D14 removes as "the run budget and `runs.json`".
   - Only Drift-Verify sets the mark today, through `ResetRuns` (`internal/driftengine/verify.go:565`). There is one mark per run, so no "drift producers" record a per-seat mark the expect seat could reuse.
   - Fix: name the record that holds the expect seat's session mark after D14, keep it apart from any judge mark, and correct "today's drift producers".

2. **MEDIUM · consistency · D3 and D13 × D14 and D15.** The in-flight `partial` rule needs the recorded skip list, which the removals take out.
   - D3 leaves out "the cases its recorded skip list names", and D13 renders "the cases left out".
   - D14 removes "`partial`, `skip` and `{skip_file}`" outright. Its read-only keep list for the `scope.json` model names only "the old compare, acceptance and waiver fields". `ScopeRecord.Skip` (`internal/driftengine/records.go:73-74`) is not on it.
   - Fix: add the scope record's `skip` field, and the `partial` constant, to D14's read-only keeps.

3. **MEDIUM · design · D1, D2, D10 and D15 × D3.** A hub edit to `diff` or `key_patterns` mid-run changes identities and mode under a sealed expectation.
   - D3 fixes the scope per run "so a run's scope does not change mid-run". D15 makes `diff` and `key_patterns` hub-only, but no decision fixes them per run.
   - A hub base edit between Drift-Expect and Drift-Verify changes D1's identities, for example `Re` becoming `#n`. The sealed `keys` entries then cover nothing, every cell they meant becomes unexpected, and D10's stored tuples stop reproducing.
   - A `diff` flip moves a run between containment by code and the opaque judge.
   - Fix: record `diff` and `key_patterns` beside the scope at Drift-Baseline, and have every later row and `check` read the recorded values.

4. **MEDIUM · design · D2 × D5 and D11.** In `command` mode the expectation and amendments are validated against identities that only exist under `lines` mode.
   - D2 exists for a capture that "cannot meet D1's contract".
   - D5's listing and validation still read D1 identities: a named case must exist, and a `keys` identity must exist unless marked `added`. D11 validates amendments "by D5's rules".
   - So the expect agent's listing, and the refusal of a legitimate amendment, rest on a line reading the capture does not support.
   - Fix: state what Drift-Expect lists and what D5 and D11 validate under `command` mode, for example cases and `why` only.

5. **LOW · consistency · D4 and its AC × D3 `off` and D12.** The "only" rule contradicts two other ways a row is Done.
   - D4 and AC D4 say either compare row "is Done at once only when the warp head equals the head of the last passing check and no amendment was recorded since".
   - D12 makes Drift-Verify and Drift-Recheck Done under a deferred capture.
   - Under scope `off` no decision says Drift-Verify or Drift-Recheck is Done. D7 would capture and check the seal with no baseline, since D3 takes no capture under `off`.
   - Fix: list the `off` and deferred cases as Done at once in D4, and add a scope-`off` row case to AC D4.

6. **LOW · consistency · D8 × D12.** D12 admits `defer` at "a Stuck … writer … row whose baseline directory is missing", but D8 never says what the `drift` entry or `lym drift check` does with a missing baseline.
   - The entry passes only under `off` or a deferral. So it is undefined whether a missing baseline fails the entry, and whether it names `lym drift defer` as D12 requires.
   - Fix: state that D8's entry and `check` report a missing counted baseline naming `lym drift defer`.

**Tool calls:** none. My context already held every file these findings cite.

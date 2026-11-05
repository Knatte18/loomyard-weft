```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. **BLOCKING — D6, D7, D8, D11 (the state with no seal or a broken seal).**
   - **Summary:** a run with no seal has no defined path, and a failed seal check has no working way forward.
   - **Case:** a darn run already at Darn-Write when this lands. It never visited Drift-Expect, so it has no `expected.yaml` and no seal.
   - **Walkthrough:**
     - D7 says such a run "is checked against an empty expectation".
     - D6 says every containment check checks the seal *before* it diffs. With no seal recorded, the check fails, and Drift-Verify is Stuck before D7's empty-expectation reading is ever reached.
     - D8 says the writer's `drift` entry is "admitted only once the seal exists". What the entry returns when no seal exists is unstated: a refusal or failure would fail the writer's gate every time it runs.
     - D6 names "a parent amendment (D11)" as the way to clear a failed seal check. D11 says an amendment "never edits `expected.yaml` or the seal". After an amendment, the hash, the committed copy and the `Warp-SHA` are all unchanged, so the seal check still fails. D11's criterion "the seal check still passes after an amendment" assumes a seal that passed before.
   - **End state:** the run is Stuck, and no verb clears it. The Open risks line on in-flight runs ("stops on any drift") describes a different outcome than the mechanism produces.

2. **MEDIUM — D1 (default key patterns on `cs` lines).**
   - **Summary:** a line with more than one quantity gets keyed by its first key only, so an expectation written against a later quantity misses.
   - **Case:** `Re = 127069, dP = 7925.267 Pa`, where the task changes only `dP`.
   - **Walkthrough:**
     - The `name = value` pattern makes the key `Re` and the value `127069, dP = 7925.267 Pa`.
     - The cell comes out as key `Re`.
     - An expect agent that writes `keys: [dP]` (the quantity it expects to move) matches no key. Containment then reports `Re` as unexpected.
   - **Purpose missed:** the record presents this line as keyed cleanly. Either the pattern must split one line into several pairs, or the expect stencil must say that a key is the first token of the line.

3. **MEDIUM — D1 (lines that change between keyed and unkeyed).**
   - **Summary:** what happens when a line is keyed on one side and unkeyed on the other is undefined, and inserting a keyed line shifts every unkeyed line below it.
   - **Case:** a hub file with unkeyed lines below a keyed block. The task inserts one keyed line at line 5.
   - **Walkthrough:**
     - Every unkeyed line from 6 on now sits one number later.
     - Pairing by line number then compares different lines at the same number. Some of those numbers hold a keyed line on one side.
     - The rules cover keyed↔keyed and unkeyed↔unkeyed only. A number that is unkeyed in the baseline and keyed in the current capture has no defined cell.
   - **Contradiction:** the D1 criterion "inserting a keyed line above another keyed line moves no other cell" holds only for keyed lines.
   - **Fix:** state that unkeyed lines are numbered by their position among unkeyed lines, or by absolute position, and add a criterion for the mixed case.

4. **MEDIUM — D13 against D7 and D10 (where the verdict's numbers come from).**
   - **Summary:** the verdict has no stored source for most of its numbers.
   - **Walkthrough:**
     - `Verdict` renders "from the drift records", and it is also called at landing (`landingdeps`), which runs no capture.
     - D7 classifies the cells but records neither the counts, the list of entries that covered no cell, nor the compared head. D4's Drift-Recheck shortcut also needs that head.
     - `judged.json` (D10) holds rulings only. It does not hold unexpected counts, reused counts or uncovered entries.
   - **End state:** the verdict section cannot be rendered as D13 specifies. D7 or D10 should name the record that stores the last check's classification and head.

5. **MEDIUM — D11 (an amendment at a Stuck Drift-Expect).**
   - **Summary:** amending at that row does nothing to unblock it.
   - **Walkthrough:**
     - Drift-Expect is Stuck when the agent's `expected.yaml` keeps failing the `expectation` entry.
     - An amendment adds entries to `scope.json`. The invalid file and the missing seal are both untouched.
     - On resume, Drift-Expect still has no valid file to seal.
   - **Fix:** admitting `amend` at Drift-Expect needs a stated effect, for example sealing an empty expectation plus the amendments. Otherwise drop Drift-Expect from the admitted rows.

6. **LOW — D6 (when the sealing commit happens).**
   - **Summary:** the seal needs a weft commit made in the middle of the producer, which is a new commit point.
   - **Walkthrough:**
     - D6 commits `expected.yaml`, then records that commit's SHA and `Warp-SHA` in `scope.json`, and that record must itself be committed.
     - Today drift records are committed only by `commitOnExit` when the producer exits.
   - **Fix:** the record should say that Drift-Expect commits twice, so the plan does not try to reuse the single commit on exit.

7. **LOW — D8 against D3 (`lym drift check` under scope `off`).**
   - **Summary:** the record gives two outcomes for the same case.
   - **Walkthrough:**
     - D8 says the entry passes under scope `off`.
     - The acceptance criterion says `check` "refuses before the seal exists", and no seal is ever made under `off`.
     - So under `off`, `check` both passes and refuses.

8. **LOW — D2 (what the judge sees in `command` mode).**
   - **Summary:** the judge rules each case on a log that shows only the first deviating line of each case.
   - **Walkthrough:**
     - The `cs` compare prints only the first deviating line per case (trial 1).
     - A `matches` ruling on that line leaves later lines of the same case unseen.
   - **Fix:** the D2 bound should name this, since it explains why containment is weaker in `command` mode.

**Each decision, the case used, and whether its purpose is met**
- **D1** — `cs` `Re …, dP …` line, and a keyed line inserted above unkeyed lines: not met (findings 2 and 3).
- **D2** — the `cs` compare in `command` mode, where it exits 1 on expected drift: met, with the bound gap in finding 8.
- **D3** — a docs run, and a seeded run carrying `partial`: met.
- **D4** — a darn run parked at Drift-Await, and a run already past Drift-Expect: met for the parked run; the run past it falls into finding 1.
- **D5** — a key entry and the open range `12-` on an unkeyed file: met.
- **D6** — an edited `expected.yaml`, and a joint edit of `scope.json` and `expected.yaml`: detection is met; the way forward is broken (findings 1 and 6).
- **D7** — an unexpected key change, and an expected entry with no drift: the classification is met; its result is not stored for D13 (finding 4).
- **D8** — the darn writer's gate after an unexpected change: met for a sealed run; undefined with no seal (finding 1) and under `off` (finding 7).
- **D9** — one expected cell with no ruling: met.
- **D10** — a matched tuple reproduced, and then a new value for the same key: met.
- **D11** — an unexpected cell at a Stuck Drift-Verify: met; not met at Drift-Expect (finding 5) or for a failed seal check (finding 1).
- **D12** — `defer` at Drift-Await: met.
- **D13** — rendering at landing: not met (finding 4).
- **D14** — no remaining reader of `baseline.md`: met.
- **D15** — the `cs` base with `compare` set and `{skip_file}` in its commands: met; the refusal and the flag together give a way forward.
- **D16** — not walked: its criterion is a review check.

I made no further tool calls.

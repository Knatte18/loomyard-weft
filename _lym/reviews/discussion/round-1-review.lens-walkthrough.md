```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on a concrete case and checked the end state against the decision's stated purpose.

**Decisions walked**
- D1, a `cs` case with `fluid = water` and `Re = 127069, dP = 7925.267 Pa` where `dP` moves: the line yields one `changed` cell on `Re`, and a keyed insert shifts no unkeyed `#n`. Purpose met.
- D2, `command` mode where the compare exits 1 and its log shows cases A and B drifted: the judge's ruling set is chosen by the judge and no code can check it. Purpose not met (finding 1).
- D3, a run seeded with `drift.scope: off` from `run_settings`: it records `off` with its source. Purpose met. An in-flight run that recorded `partial`: the outcome is undefined (finding 4).
- D4, tapestry where Drift-Verify passes at head H and the perch commits nothing: Drift-Recheck is Done at H. Purpose met. A regression added after the writer: no row can fix it (finding 3).
- D5, the expect agent lists `Re` with a `why` for the line above: the parser accepts it because `Re` is in the identity listing. Purpose met.
- D6, a darn run (Record-Write → Drift-Baseline → Drift-Await → Drift-Expect): the seal commit's `Warp-SHA` equals the baseline head, because no row between them moves the warp. Purpose met.
- D7, an unexpected `dP` move: the row goes Stuck naming case, identity, old and new value. The detection purpose is met; the way forward is not (finding 3).
- D8, tapestry where the `drift` entry fails on Master's run-level gate: Master edits no code and the entry has no fixer. Purpose not met (finding 2).
- D9, `lines` mode with two expected, unruled cells: the judge rules both and the `judgement` entry checks ids and open ranges. Purpose met in `lines` mode. The open-range statements are lost afterwards (finding 5).
- D10, a reproduced tuple after an unrelated commit: its stored ruling is reused and no judge is spawned. Purpose met.
- D11, Drift-Verify Stuck on unexpected cell X, then the parent amends key X and the run resumes: a full pass runs, X is expected, the judge rules it, Done. Purpose met.
- D12, a counted capture whose baseline directory is gone: Stuck, then `defer`, then both compare rows Done and the verdict states the deferral. Purpose met.
- D13, a verdict for a run with one open range: it renders the range and its `why`, but not what the judge checked under it (finding 5).
- D14, removals: no mechanism to walk. Held.
- D15, the `cs` hub base with `compare` set and `diff` unset: the reinterpretation flags it, and the capture holding `{skip_file}` is refused naming the edit. Purpose met.
- D16, the darn expect agent reads the decision record: Record-Write wrote it before Drift-Baseline (`contracts/recipes/darn-recipe.yaml:74-76`). Purpose met.

**Findings**

1. **MEDIUM · design · D2 and D9, `command` mode.** Nothing checks that the judge rules every drifted case, so a judge that omits a case, or writes no rulings, ends Done.
   - The compare is opaque, so only the judge reads which cases drifted. The `judgement` entry can only check "every case has a valid ruling" against the cases the judge itself chose to list.
   - Example: the log shows A and B drifting and the judge writes only `A: matches`. No ruling is other than `matches`, so nothing goes Stuck.
   - D2's Bound names the log's first-deviating-line limit but not this one.
   - Evidence: `_lym/discussion/decision-record.md:68-71`, `:188`, `:190`, `:193`.

2. **MEDIUM · design · D8, tapestry.** The premise "the entry carries no cold fixer: the writer, who holds the context, fixes its own regressions" is false for tapestry.
   - The Webster row's `gates` list reaches Master's run-level gate (`internal/websterengine/runlevel.go:790`).
   - Master's stencil says "You never edit code yourself" and "Do NOT localize or fix the failure yourself". Its only fix paths are a fixer fork or `fix-verify`, both driven by the verify gate's own prompt file (`contracts/stencils/webster/webster-template-master.md:20`, `:192-205`).
   - The forks that wrote the code are gone by then. A failed `drift` entry re-prompts a session that cannot act on it, and the D16 paragraph added to Webster's stencil tells Master about a check it cannot answer.
   - Evidence: `_lym/discussion/decision-record.md:173-174`, `:283`.

3. **MEDIUM · design · D4, D7 and D11.** A real regression introduced after the writer leaves the run Stuck with no way forward that fixes the code.
   - Code-Burler-Fix edits code right before Drift-Recheck (`contracts/recipes/darn-recipe.yaml:192-226`); in tapestry, Parent-Align's merge fixer runs before Drift-Verify.
   - Such a regression stops Drift-Verify or Drift-Recheck with an unexpected cell.
   - None of the available exits fixes code:
     - Drift-Recheck loses its route back.
     - Drift-Verify spawns no code-editing agent.
     - `amend` only adds coverage, which would accept the regression.
     - `defer` is admitted only for a missing baseline.
   - The decision names no way forward for this case.
   - Evidence: `_lym/discussion/decision-record.md:100`, `:108`, `:160-161`, `:219`, `:225`.

4. **MEDIUM · design · D3, in-flight runs with a recorded `partial` scope.** D3 states what happens to a recorded `off` or `full`, but not to a recorded `partial`.
   - That run's baseline capture skipped the listed tests: the skip file was written and those case files are absent (`internal/driftengine/capture.go:39-40`, `:91`).
   - Every later capture is full, so each skipped test diffs as an `added case`. Its run has no seal, so the expectation is empty, those cells are unexpected, and the run goes Stuck at Drift-Verify.
   - Open risks covers only a run past Drift-Expect, not this cause.
   - Evidence: `_lym/discussion/decision-record.md:82-84`, `:316`; `internal/driftengine/records.go:20-21`, `:73-74`.

5. **LOW · consistency · D9, D10 and D13, the judge's open-range statements.** The judge's `open_ranges` list ("a sentence on what the judge checked under it") is written and gate-checked, then has no reader.
   - D10 stores only ruling tuples.
   - D13 renders each open range with the expectation's `why`, not what the judge checked.
   - `judgement.yaml` is overwritten by the next judge.
   - So operator decision 2's "listed in the judge's verdict" never reaches the pull request.
   - Evidence: `_lym/discussion/decision-record.md:187`, `:200`, `:242`.

**Tool calls**
- Read `contracts/recipes/darn-recipe.yaml`'s row order and record producer: the manifest read only tapestry's rows, and D6 and D16 depend on darn's order.
- Listed `contracts/stencils` to find Master's stencil: needed to check D8's "the writer fixes its own regressions" against tapestry's run-level gate session.
- Grepped `contracts/stencils/webster/webster-template-master.md` for its code-edit and gate-failure rules: same reason, D8's premise.

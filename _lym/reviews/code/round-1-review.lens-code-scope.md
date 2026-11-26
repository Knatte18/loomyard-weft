```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to a board-entry goal point or the Added entry, and each decision to the changes that build it.

**Decisions** (board entry `## Goal` items 1–8, plus the one `Added after Discussion` entry)

- **D1: hardener findings exempt from the changed-file and changed-hunk rules.**
  Built by C1. C1 also adds an exemption from the BLOCKING-only rule, which this item does not name (finding S1).
- **D2: FAIL blocks.** Built by C2.
- **D3: one convergence rule for a MEDIUM first raised in the closing round.** Built by C3.
- **D4: kill rule.** Built by C4.
- **D5: `resolved` needs a whole repro.** Built by C5.
- **D6: the fix half drives the neighbours of its fix.** Built by C6.
- **D7: the verdict lists what no sandbox can drive as follow-ups.** Built by C7.
- **D8: one opus round.** Built by no change. The Added entry asks for none, and `internal/hardenshed/template.yaml` is untouched. Correct.
- **Added entry (item 8 is a per-campaign setting).** Honoured: no code change, and the item-7 wording "a follow-up for the run's parent to file" matches the entry's text.

**Changes**

- **C1.** The `RoundContext.HardenerRound` exemption in `internal/burlerengine/roundcheck.go` (`carriedFinding`, `newFindings`). Traces to D1.
  - The wiring rename `FocusFindings` → `HardenerRound` in `burlercli/validate.go`, `shedadapters/burlerreview.go` and `roundcontext.go`, and `shedrecipe/entries_burler.go` is forced by C1.
  - So are the `burlerengine/doc.go` text, the removal of the `FocusOrigin` constant (its last use went with the old exemption), the `burler-review-format.md` and `burler-step-review.md` bullets, and the test rows in `roundcheck_test.go`, `roundcontext_test.go` and `entries_burlerhalves_test.go`.
  - Commit `ed9dd0945` only rewraps comments and is forced too.
- **C2.** The rubric's FAIL bullet (`harden-rubric-review.md:59-60`, and "undriven, WARN or FAIL" in Converged). Traces to D2.
- **C3.** `BouncerConfig.FirstRaisedGates`, `decisionRuleMarker`'s new parameter, `bouncerEntry` setting it for `harden_round`, the `shedadapters/doc.go` paragraph, and the rubric's Converged bullet and its "Class each fault … `design`" rule. Traces to D3.
  - The class rule is needed to make the rubric and the Bouncer state one rule, so it traces as well.
  - The test changes in `bouncerprompt_test.go` and `bouncer_seed_test.go` are forced.
- **C4.** The "Kill every job both ways" bullet. Traces to D4.
- **C5.** The two lines in "Re-drive earlier fixes". Traces to D5.
- **C6.** The neighbours instruction in `hardener-recipe.yaml`'s fix row. Traces to D6.
  The entry says "every role"; the diff says "every role and verb". That is within the item.
- **C7.** The follow-ups sentence in Coverage. Traces to D7 and the Added entry.

No change traces to nothing, and every decision is built.

**Findings**

- **S1 — LOW, scope, `internal/burlerengine/roundcheck.go:309-312`**
  - **Summary:** the diff also lifts the incremental round's BLOCKING-only rule for every new hardener finding. Goal item 1 names only the changed-file and changed-hunk rules.
  - **Evidence:**
    - The board entry's item 1 (`lym board get`, `## Idea` item 1) names the changed-file and changed-hunk rules only.
    - The `HardenerRound` `continue` sits before the `f.Severity != SeverityBlocking` check (`roundcheck.go:309-316`).
    - `burler-review-format.md:78-79` and `burler-step-review.md:55` restate the lift.
    - The summary's "What changes" claims it as part of item 1.
  - **Mitigation:** item 3 presupposes "a MEDIUM first raised in the closing round". Every hardener round from round 2 is incremental (`internal/shedadapters/roundcontext.go:218`, `Incremental: !generation.Amendment`). Under the old rule such a MEDIUM could not exist outside `origin: parent`. So the lift is arguably forced by D1 and D3 read together. LOW because of that implication. The fix is to record the reading with the parent, or keep the BLOCKING rule.
- **Not a finding: the class-gating choice.** Choosing `design`-only gating means a MEDIUM `consistency` or `scope` finding no longer blocks a hardener CONVERGED, which the old rubric's "no BLOCKING or MEDIUM finding" rule did block. D3 asks for one rule and leaves the choice of rule open. The summary's "Design decisions" names the choice, so it is within D3's licence.

Tool calls:
1. `grep Incremental` in `roundcontext.go` and the `shedrecipe`/`hardenshed` packages. My context lacked where `RoundContext.Incremental` is set, which decides whether hardener rounds from round 2 are bound by the BLOCKING-only rule.

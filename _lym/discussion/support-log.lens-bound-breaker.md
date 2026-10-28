```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens on the validator-checks decision record: I attacked each claim and bound it states. Seven claims hold and seven break or have a gap.

**Findings**

1. **BLOCKING — D4 (and D1, D2, D10 text): the glyph rule catches markdown headings.**
   - D4 treats any backticked token that holds `#` and has no `plan:` prefix as a glyph and resolves it.
   - That catches backticked headings, which records cite all the time. This record has `` `## Acceptance criteria` ``, `` `## Decisions` ``, `` `### D<n> — <title>` `` and `` `### Added after Discussion` ``.
   - Other tokens hit it too, such as `` `C#` `` and `` `#123` ``.
   - None of these resolve, so each one is a `citation-symbol-missing` finding. Marking it ` (new)` would be wrong.
   - Counterexample: this record, once the check exists, fails on its own D1 and D10 lines.
   - The token rule also says nothing about whitespace. A call token such as `` `loomshed.ValidatePlan(plan, root)` `` holds spaces and nested parentheses.
   - Fix: a token is a glyph only when it has no whitespace and parses under quarry's glyph grammar (`<unit>#<member>`, unit not empty). Any other token is not tested.

2. **MEDIUM — D8 and D6: the "list it under `Uses`" answer collides with the existing `uses-later-target` check.**
   - `planparser.checkUsesLaterTarget` refuses a card whose `Uses` entry is a target of a card with a higher number.
   - Counterexample: card 2 edits code in package P and lists `P/doc.go` under `Uses`, as D8 tells it to. Card 5 edits `P/doc.go`. The result is a blocking `uses-later-target` on card 2.
   - D8 says the `Uses` edge "orders the card after a card that edits that doc". That only holds for an earlier editor; a later editor gets a refusal.
   - "The cost is one `Uses` bullet" is also false. The writer must reorder the cards or list the doc under Edit or Extends on card 2.
   - The same breaks D6 when a later card edits the seam test.
   - Fix: state this interaction and its way forward, or exempt these listings from `uses-later-target`.

3. **MEDIUM — D6: the bound leaves out cards that do not name the struct.**
   - D6 fires only when a target names the struct or one of its members.
   - Missed today: a card that adds a field through a file target such as `internal/loomcli/landingdeps.go`, or a package self glyph such as `internal/loomcli#`.
   - A field created through a `plan:` handle (`plan:internal/loomcli#Deps.X`) matches only if the `plan:` prefix is stripped, and the record does not say it is.
   - The stated bound covers only method changes.
   - Fix: state the file and package miss as a bound or cover those targets, and say how handle spellings match.

4. **LOW — D3 and D4: one writer-uneditable part is not exempt.**
   - The exemptions cover `Added after Discussion` entries and carry-over blocks.
   - They miss `## PATTERN changes` bullets that carry the ` — added after Discussion` suffix. The spec says no writer edits those.
   - Counterexample: `lym loom decision add --pattern-text` names a new `pattern/PATTERN-x.md` during Discussion review. From then on the loom gate refuses a record the fixer cannot fix.

5. **LOW — D4: only two quarry answers are defined.**
   - D4 covers resolved and not-found, but quarry also answers ambiguous, multipart and rejected.
   - planglyph counts an ambiguous answer split by build constraint (for example `foo_linux.go` and `foo_windows.go`) as resolved. D4 does not say whether the discussion check uses planglyph's status policy.
   - Fix: say the check uses the same status policy as `glyph-not-found`.

6. **LOW — D4: new dependency on quarry not stated.**
   - The Discussion-Write and Discussion-Burler-Fix gates now need quarry.
   - A quarry outage halts Discussion as an infrastructure error, where today it passes. That is correct ("never a pass"), but D4's bound should name it as a new way for Discussion to halt.

7. **LOW — D5: two existing helpers fall outside the scan.**
   - `assertNoZeroFields` in `internal/fabricengine/mergestate_integration_test.go` and `assertStructFieldsSideFree` in `internal/fabricengine/mergevocab_test.go` reflect over whatever struct they are handed.
   - The tests that call them are seam tests of record structs, and the scan never sees them.
   - The general helper bound covers this, but the acceptance line "today's reflective tests carry markers" reads wider than the scan can hold.

8. **LOW — D5: marker placement next to `//lym:guard` is not settled.**
   - PATTERN-test-speed puts `//lym:guard` on the line directly above `func Test…`.
   - `impactset`'s `guards_test.go` treats a stray line between the guard and the func as a defect; only `//testtiming:keep` is allowed there.
   - A test that carries both `//lym:guard` and `//lym:seam` therefore needs a parser rule the record does not state.

9. **LOW — Open risks: darn runs are missing.**
   - In-flight darn runs also meet D1 and D2 at their handoff gate.
   - The risk line names only loom runs.

**Claims attacked**
- **Broken:** findings 1 to 9 above.
- **Held:**
  - `lym loom decision add` runs `Validate` alone (`append.go:93`).
  - `ValidateLoomDiscussion` has only two callers, the gate and the verb.
  - No two Go package directories share a base name today, so `citation-symbol-ambiguous` cannot be reached yet and its acceptance line needs a fixture.
  - Dispatch never runs `planGatePass` (planglyph doc).
  - The `Added after Discussion` heading prefix is excluded from D1.
  - D2's whole-word match keeps `D10` from counting as `D1`.
  - The `_lym` and `.lym` exclusion through `lymdirs` holds.
  - D7's skip when `gate.yaml` is absent holds.

**Tool calls**
1. Grepped `append.go`, the callers of `ValidateLoomDiscussion` and `checkUsesLaterTarget`, to test the decision-add claim, the wiring reach and the `Uses` interaction; my context did not have these function bodies.
2. Ran `find` for Go package directories that share a base name, and read the `NumField` site in `mergestate_integration_test.go`, to test D4's ambiguity rule and D5's scan reach; my context had no directory census and no view of that site.
3. Read the `NumField` sites in `mergevocab_test.go` and `websterengine/config_test.go`, plus AppendDecision's PATTERN-bullet path, to test the D5 scan and the D3/D4 exemptions; my context had only grep line numbers for these.

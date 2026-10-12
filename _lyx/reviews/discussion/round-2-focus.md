---
round: 2
focus:
  - "Decision 5 (overlay): the plan check path calls CanonicalizeHandles, which calls planparser.RewriteRefs, which re-parses the plan from disk and writes card files, and resolvePass then reloads with planparser.ParsePlan from disk. Settle whether the decision record bounds an overlay run so that it writes no plan file and every later check reads the overlaid plan, including the reviewer's flag-less baseline run, and whether an acceptance row covers an overlay adding a non-canonical plan handle."
  - "Decisions 1 and 4: ValidateFormatAfter, once the run record holds a done card, runs ValidateDispatch (scan-only delete-before-reference) and then the typed planGatePass over pending cards. Settle whether the record states which entry points run which form, so a later-region method call is reported once at the plan gate, and whether an acceptance row covers a plan gate with a done card."
  - "Decision 4: laterDeleteReferenceReasons feeds both batch recording's refusal and recover-batch's ErrRecoveryDeleteReferenced refusal. Settle whether the record states recover-batch's behaviour on informational name-only findings and on blocking package-level ones, in Decisions, Scope and the acceptance rows."
  - "Decision 5 name rule and refusals: settle whether an overlay of a top-level .md file that ParsePlanFrom never reads (amendments.md or an orphan file) is refused with a way forward, and whether terminal stdin appears in the refusal acceptance row."
  - "Decision 3: caller-uncovered's candidate walk skips Go files under testdata, vendor and directories starting with . or _. Settle whether the Decision states or bounds that skip for delete-before-reference, and whether an acceptance row covers a package self glyph Delete's import-path match being limited to .go Edit regions."
  - "Decisions 6 and 7: settle whether Decision 6 states a procedure for finding the offending fix that handles two fixes producing a blocking finding only together, and whether Decision 7 says what the reviewer names when no member span encloses the cited line."
---

Each entry names where to look and the question to settle; none pre-states a verdict.

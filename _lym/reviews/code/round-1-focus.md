---
round: 1
focus:
  - "contracts/stencils/burler/burler-step-fix.md, Parent ruling rule: it says the fixer's parent directive names both ruling kinds (parent mail, decision notice). Check the fixer's parent directive stencil or code names both, and that the rule's wording covers all three asks of the board entry: override, a Rulings report section, a way to flag a wrong ruling."
  - "burler-step-fix.md and burler-template-fixer.md: the rule says to flag with `lym creel post --parent` and no `--expect-reply`, and the seat's no-message rule admits that one flag. Check the two stencils and the fixer's allowed commands agree with each other and with the creel verb's real flags and refusal behaviour."
  - "internal/burlerengine/prompt.go routedDecisionsLine: it now points at 'the parent ruling rule below'. Check that the line is rendered where that rule is reachable, that every caller and test of the constant still holds, and that no other prompt text still says the fixer leaves decision findings out unconditionally."
  - "internal/burlerengine/doc.go and the stencil header comment: check the new prose matches what the stencils and code do, and whether anything that reads or validates the fixer-report (repairReportBeforeGate, report parsing, gates) must accept or require the new Rulings section."
  - "Test economy: the diff adds no test. Name any behaviour added in code (not stencil prose) that has no coverage; do not ask for assertions on stencil prose."
---

The diff changes two stencils, one constant and one package doc, answering board entry #84.

# Support log — webster-deviations-list

## Interview

### Turn 1 — autonomous exploration

No operator answers this session; every pick below is an auto-pick.
Read the board entry, the board list, the implementer body, the fork prefix, the locate and guard-fix fragments, the Master template's digest section, `ScopeGuard` and `targetsCover`, `websterengine/doc.go`'s fork-return contract, the plan spec's deviation paragraph, and commit `30385e1ae`.
The `kickstart-per-card` fragments under `contracts/stencils/webster/` carry no deviation or union wording.
Finding: `30385e1ae` (2026-10-13) moved the deviation list from the fork's report to the scope guard, so the fork-side contradiction the note recorded is gone; the Master template already reads the digest's guard-filled list.
What remained was the body's "every other deviation rule stands", which points at rules the body does not state, and the plan spec's stale description of the comparison.

### Turn 2 — first design

Picked one rule in step 2 of the per-card loop matching `ScopeGuard`, and a pointer to `ScopeGuard` in the plan spec, with no Go change.

### Turn 3 — lens review

Six forks reviewed the first record; all returned.

- premises, MEDIUM: `ScopeGuard` matches symbols only exactly against the target union, so members of a card's own file or package targets are reported; the first record's rule matched that literally and banned the card's own edits. Fixed: verified through `resolveKeyFor` and `RefScope`, and added D1, the guard fix, which is the board entry's bug as the guard now produces it.
- premises, LOW: the plan spec paragraph sits under `## Deferred / forward-compat` beside an existing pointer to `internal/websterengine`'s doc. Fixed in D3: merge with that pointer.
- goal-scope, LOW: the plan spec edit is not in the board entry. Rejected: it is the other text defining the same comparison, and it is stale; the scope line says why.
- goal-scope, LOW: renames missing from the rule. Fixed in D2.
- goal-scope and scenarios, LOW: the verify-fix body not named. Fixed under Scope Out.
- goal-scope, LOW: the post-`kickstart-per-card` fragment check not recorded. Fixed in Turn 1.
- board-fit, MEDIUM: `card-own-entry-edit` may edit the plan spec too. Fixed under Open risks.
- board-fit, LOW: `weaver`'s open deviation question. Fixed under Scope Out.
- board-fit, LOW: F13 of the audit may touch the body. Rejected: F13 cites `pattern/pattern-directive-implementer.md`, not the body.
- pattern-fit, MEDIUM: deployed spec copy. Fixed under Constraints.
- pattern-fit, LOW: why the stencil restates while the spec points. Fixed in D2's rationale and Constraints.
- pattern-fit, LOW: name the pinned Master digest-field set. Fixed under Constraints.
- scenarios, MEDIUM: the guard-fix fragment's scan-error fix is a third sanctioned deviation. Fixed in D2.
- scenarios, MEDIUM: the self-fix exit's bound unstated. Fixed in D2: the two-pronged test and the attempt cap.
- scenarios, LOW: no criterion that a deviation never makes the report `FAILED`. Fixed under Acceptance criteria.
- scenarios, LOW: resume and first batch unaffected. No change needed.
- leanness, MEDIUM: D1's history belongs in the log. Fixed: the history is in Turn 1, and the old D1's requirement moved to Scope Out.
- leanness, LOW: Auto-mode assumptions repeat Scope and D3. Fixed: reduced to a pointer to the ledger.
- leanness, LOW: D3's rationale restated the guard's internals. Fixed.
- leanness, LOW: the recovery note stated twice. Rejected in part: D2's rationale keeps one clause because it is why the body states the rule once; the Notes bullet tells the plan writer no recovery edit is needed.
- leanness, LOW: the Master template exclusion's reason. Fixed: the reason is in Turn 1.

## Rejected alternatives

- Close the task as already fixed with no edit: the guard still reports members of a card's own file and package targets, and the body refers to deviation rules it never states.
- Keep `ScopeGuard` as it is and only state its rule: the rule would tell a fork that edits inside its own file target are deviations, the opposite of step 2's "exactly the targets its type labels name".
- Let a member glyph target cover its whole package, as the symbol-less file check does: a card naming one function would hide edits to every other member of its package.
- Restore a fork-side deviations field: the guard already computes the list from the delta; a fork-side list could only disagree with it.
- State the full comparison in the plan spec too: a second restatement goes stale; a pointer to the code does not.
- Put the rule in a new stencil fragment: the body is already shared by fork and recovery prompts.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): change `ScopeGuard`? Yes, after the premises lens: file and package targets cover their members, member targets stay exact.
- Q2 (auto-pick): change the fork report or digest? No: both already match the design.
- Q3 (auto-pick): where does the stencil rule live? Per-card step 2 of the implementer body, beside the existing "exactly the targets" sentence.
- Q4 (auto-pick): does the plan spec paragraph belong in scope? Yes: it is the other text defining the comparison and is stale against `ScopeGuard`.
- Q5 (auto-pick): tests? Extend `ScopeGuard`'s existing tests; no test pins stencil prose.
- Q6 (auto-pick): touch F12's report-reader wording? No: it stays with the `stencil-intent-audit` note.

# Card 8 — The implementer reports no deviations, and Master reads the guard's list

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils#wordingClaims`
- `contracts/specs/loom-plan-spec.md`

**Intent:**
D4's stencil half, after card 7 made the guard the list's only source.

`webster-body-implementer.md`'s report section asks for `status` and `head_sha` only: the YAML block drops `deviations:` and its `- <path>` line, "these three fields" becomes "these two fields", and the paragraph explaining `deviations` is cut, leaving the `status` and `head_sha` sentences and the rule that only a failed card gate, card `verify:`, batch gate or broken test makes `status` `FAILED`.
Its `## Prior-batch context` paragraph says the predecessor line's `scope-guard deviations` are the files record-batch's scope guard found changed outside that batch's targets.

`webster-template-master.md`:
- the digest field list keeps `deviations`, and its explaining line says the list is the files record-batch's glyph scope guard found changed outside the batch's own targets, informational and never a reason a batch is `stuck` on its own;
- the summary paragraph says a fork's whole report is `status`/`head_sha`, and that the narrative at `{{.summary_path}}` names each batch's deviations from the digest's list, the guard's result, instead of "any reported deviations from the plan's declared file-ops".
It points at the digest and never restates the guard's comparison (`PATTERN-producer-pointer-rule`).

`wordingClaims` in `contracts/stencils/claims_test.go`:
- the implementer entry's `wantAll` "the report carries the minimal fork-return contract's keys" drops `"deviations:"`, keeping `status:` and `head_sha:`, and a `wantNone` "the implementer asks for no deviation list, which the scope guard alone produces" pins `"deviations:"` and `"deviation union"` absent;
- the Master entry gains a `wantAll` "Master reads the deviation list from the digest, with the guard as its source" of "scope guard" and "the digest's list", and a `wantNone` "the summary narrative no longer reads fork-reported deviations" of "any reported deviations from the plan's declared file-ops" and "`status`/`head_sha`/`deviations`".

`internal/websterengine`'s template test pins the Master's digest field names, `deviations` among them; the list keeps that name, so the test is unchanged.

`contracts/specs/loom-plan-spec.md`'s paragraph on the `changes-files`/deviation union drops "a fork reports `OK, SHA <x>` or a deviation note" and says the fork reports `OK` and a SHA while record-batch's mechanical guard produces the file list, informational as before.

**ImpactSummary:** Two webster stencils, their pinned phrases and one spec paragraph; no Go behaviour changes.

**Commit:** 8: Drop the fork's deviation list from the webster stencils

---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/configreg/configreg_test.go:204
    summary: keyListWayForward and keysOutside carry the add, drop and sort-only message logic the Idea asks for, and no test pins any branch
    fix: In internal/configreg/configreg_test.go, add one tier-0 table test TestKeyListWayForward that opens with tiergate.Require(t, 0) and t.Parallel(). Rows are add only (got has a key want lacks), drop only (want has a key got lacks), add and drop together, and order-only (same keys, different order, expecting the sort message). Assert the exact returned string for each row.
  - id: F2
    severity: NIT
    class: consistency
    origin: handler
    location: internal/configreg/configreg_test.go:202
    summary: The keyListWayForward doc comment is garbled ("the test's expected list, named list, so it matches got")
    fix: Reword the first doc-comment line of keyListWayForward to "keyListWayForward names the keys to add to and drop from the expected list called list in this test, so that it matches got." Keep the second line.
---

### [LOW] keyListWayForward and keysOutside have no test

**Location:** internal/configreg/configreg_test.go:204-227

**Issue:** The decision record asks that the classification test's failure message say which key to add where.
The diff adds that through `keyListWayForward` and `keysOutside`, which pick between three outputs: add, drop, and sort-only.
Only a wrong classification reaches them, so a flipped direction or a wrong list name would pass unnoticed.
The rubric's test-economy bullet lists a behavior the diff adds with no coverage.
The public classification test cannot reach these branches without a broken registry, so a direct table test is the only way to pin them.
The code is correct as written: the add and drop directions match got against want, and an order-only mismatch yields the sort message.

### [NIT] keyListWayForward doc comment is garbled

**Location:** internal/configreg/configreg_test.go:202

**Issue:** The comment reads "names the keys to add to and drop from the test's expected list, named list, so it matches got".
The phrase "named list" does not parse.
It should say that `list` names the expected list in the test.

## Rejected

- Missing `//lym:seam internal/loomengine#Config` on `TestLoadConfig_Loads` (lens:code-affected-set F1): the `template defaults` subtest looks fields up by name and does not walk the field set. The test already compared the whole `Config` with `reflect.DeepEqual` before this diff, with no marker, and no mechanical scan flags it.
- `MachineKeys` failure message reads awkwardly with two clauses (lens:code-affected-set F2): the text is correct and carries the way forward.
- `specFunc` is blank for a nil `Spec` (lens:code-tests F2): a nil `Spec` already panics at `m.Spec()` in the same subtest, and every registered `Spec` is a named package function.
- The decision record's "list the other tests that fail for any added key" is met only in `_lym/landing/summary.md`, not in the diff: the Idea asks for a list, not a code change, and the summary holds one. The spot-checked entry (`internal/darnengine/config_test.go`, a `templateConfig()` literal) bears out its claim.
- Fields the old `templateConfig()` left at zero (`PRReviewAskParent`, `Seed`, `DiscussionAdvisors` and the judge keys) are no longer pinned: this follows the Idea, where each case asserts only the fields it is about.
- No case is vacuous: where a mutate sets a field to the template's own value, a sibling field in the same case changes. The `template defaults` subtest covers the same 41 fields as the removed literal and fails loudly on a renamed field.

I also ran `lym gate test ./internal/loomengine ./internal/configreg`: tier0 passed 603, failed 0.

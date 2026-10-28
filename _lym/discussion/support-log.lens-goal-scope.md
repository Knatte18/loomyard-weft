```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the decision record against the `validator-checks` board entry.

**Decision notes:** the `lym board list` output already in context lists no note titled `Decision for validator-checks:`. I read none, and none was out of reach. The record has no `Added after Discussion` entries.

**Goal points of the entry:**
1. Citation check, that every cited repository path and Go symbol exists: partly met (D3, D4). Unqualified symbols are left untested, and darn records are not checked. Findings F1 and F2.
2. Acceptance check, that every decision id appears in an acceptance criterion: met (D2). D1 adds the id heading the check needs, which counts as licensed groundwork.
3. Seam-test check, for a card that adds a field to a shared `Deps` or `Env` struct: met (D5, D6), but the scope is wider than the entry. Finding F3.
4. Tier-tag check, that a new test file states exactly one tier tag: met with relaxations (D7). Finding F4.
5. `doc.go` check, for a card whose Intent changes a package's behaviour: met through a proxy (D8), with the reason stated.
6. Each refusal names what it found and the fix: met (D1 to D8 each give a way forward).
7. The writer stencils state the new rules: met through D10's pointers, with the `PATTERN-stencil-ownership` conflict recorded under Constraints. This is legitimate because the prompt says PATTERN wins.
8. Out of scope (lens and profile changes, design-interaction and unbounded-edge checks): respected. Nothing in the record touches either.

**Findings:**

- **F1 — MEDIUM, `consistency`, D4 and Scope.** The entry asks that every Go symbol the record cites exists. The record tests only glyphs and package-qualified names, and leaves unqualified names untested. Evidence: the entry says "every repository path and Go symbol the decision record cites exists". D4's bound and Scope's Out line ("bare unqualified names such as `MessageLine` or `Stop`") narrow that. The narrowing depends on a spec rule that writers qualify symbols, but nothing machine-checks that rule. Fix: either refuse the unqualified Go-symbol shapes the check can recognise (such as `Type.Member` with an upper-case owner), or state plainly in Scope that this narrows the entry's "every".
- **F2 — MEDIUM, `scope`, D4.** The entry places the citation check in `validate-discussion` "and the gates of the rows that run it", and on a darn run that verb is the Darn-Write gate check. The record leaves darn out. Evidence: the entry's Design section names no recipe exclusion. D4's reason ("darn has no Discussion review round to save") is the writer's own judgment and nothing in the entry licenses it. The writer can settle this: either run the check on darn too, with the ` (removed)` mark covering code already landed, or keep the exclusion and list it under Auto-mode assumptions as a narrowing of the entry. Today Auto-mode assumptions records the pick but not that it departs from the entry.
- **F3 — LOW, `scope`, D5 and D6.** The entry scopes the check to a "shared `Deps` or `Env` struct". D5 marks every test that reflects over any struct's fields, config structs included. Evidence: the reflective loops in `internal/loomengine/config_test.go` and `internal/fabricengine/mergevocab_test.go` sit in the writer's exploration. Under D6, any card that touches those structs must then list those tests. The extra reach is unlicensed and unstated in the record. Fix: say so in D5 as a deliberate widening, or restrict the marker to seam structs.
- **F4 — LOW, `consistency`, D7.** The entry says "exactly one tier tag". D7 also accepts a `||` disjunction and `untagged`. Evidence: the entry's line versus D7's text. The reason (`PATTERN-test-speed` allows both) is sound, but the departure from the entry's wording should be stated in D7 or under Auto-mode assumptions. Auto-mode assumptions does name it, so this is close to settled; only D7's own text omits it.

No `decision`-class doubts: the task's purpose matches the entry and the audit report it cites.

Tool calls made: none. My context already held the `lym board list` and `lym board get` output for this task, and the list showed no decision notes.

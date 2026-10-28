```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the decision record against the `validator-checks` board entry, its decision notes and the work that has landed since.

**Goal points of the entry**
- Citation check in `validate-discussion` (every repository path and Go symbol the record cites exists): met with narrowings. D3 and D4 do it on loom only. Unqualified names, lower-case members and outside packages are left untested. Scope's Out list and the Auto-mode assumptions state these narrowings.
- Acceptance check in `validate-discussion` (every decision id appears in at least one acceptance criterion): met by D1 and D2. D1's heading form is the id this check needs.
- "Gates of the rows that run it", discussion side: met. D2 names the loom and darn gates and the verb. D4's loom-only citation check is listed as a narrowing.
- A card adding a field to a shared `Deps` or `Env` struct lists that struct's reflective seam tests: met and widened. D5 covers every reflected struct and D6 holds any target that names the struct. Both widenings are listed under Auto-mode assumptions.
- A card adding a test file states exactly one tier tag: met with relaxations. D7 also accepts `||` and `untagged`. D7's own text gives the reasons, but Auto-mode assumptions does not list them (finding 2).
- A card whose Intent changes a package's behaviour lists that package's `doc.go`: met through a proxy, D8, listed under Auto-mode assumptions.
- "Gates of the rows that run it", plan side: met by D9.
- Each refusal names the item and what the writer adds: met. D1, D2, D3, D4, D6, D7 and D8 each state a way forward.
- The writer stencils state the new rules: met as pointers (D10). The Constraints section records that `PATTERN-stencil-ownership` wins over the entry's wording.
- Out of scope (lens and profile changes, the design-interaction and unbounded-edge classes): respected in Scope's Out list.
- Decision notes: `lym board list` shows no note titled `Decision for validator-checks:`, so none was read and none was out of reach. The record has no `Added after Discussion` entries.

**Findings**

1. LOW, consistency. Location: `decision-record.md`, `## Open risks`, second bullet.
   Summary: the bullet treats `landing-settles-in-place` as a peer that has not landed yet. It landed on `main` as 71a46b11 ("Landing merges a moved parent in place…", 2026-10-28), which is not in this branch's HEAD.
   - That commit changes `internal/loomcli/landingdeps_test.go`: it removes `Deps.LargeMergeLands`, which the test's `unfilled` map names.
   - It adds no new `NumField`, `VisibleFields` or `NilSeams` call.
   - The writer can settle the bullet by citing the landed commit: the marker goes on the test as it reads after 71a46b11, and no other reflective test was added. Otherwise drop the bullet.
   - Evidence: `git merge-base --is-ancestor 71a46b11 HEAD` reports not an ancestor; `git branch --contains` shows `main`; the commit's stat lists `internal/loomcli/landingdeps_test.go`; the grep of its added lines found no reflection calls.

2. LOW, consistency. Location: `decision-record.md`, `## Auto-mode assumptions`, against D7.
   Summary: this section is meant to keep "only the readings of the entry" (support-log Turn 5, leanness 1). It lists the readings behind D4, D5, D6 and D8, but not D7's reading of "exactly one tier tag" as "one tag, or a `||` disjunction, or `untagged`". That is the same kind of departure from the entry's wording, so it should be listed too. The fix is one bullet under Auto-mode assumptions.
   - Evidence: the board entry body says "states exactly one tier tag for it". D7 is the record's `### D7` section.

**Tool calls**
- `lym board list` (filtered): to find decision notes for this task and the peer named in Open risks; my context had only the entry body.
- `git log --grep` for `landing-settles`: to check whether the peer in Open risks had landed.
- `git branch --contains` and `git merge-base`: to place commit 71a46b11 relative to this branch.
- `git show 71a46b11` (grep of added lines for reflection calls): to see whether the landed peer added a reflective test D5 would need to mark.
- `git show 71a46b11 -- PATTERN.md`: to check whether the landed commit touched `PATTERN-test-speed`, which this task declares a change to. It did not; it changed `PATTERN-shed-stop-chain`.

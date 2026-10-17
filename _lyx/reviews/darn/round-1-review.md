---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/burlerengine/doc.go:293
    summary: The fork-discipline paragraph still says forks run no git commands of any kind, contradicting the paragraph this diff adds to the same doc.go
    fix: In internal/burlerengine/doc.go, in the paragraph that begins "Fork discipline is fixed boilerplate", replace "no git commands of any kind" with "git commands that read only, such as git diff, git log and git show, and none that changes the repository, its index, its refs or its working tree".
---

### [LOW] doc.go still states the retired no-git fork rule

**Location:** internal/burlerengine/doc.go:293
**Issue:** The diff changes `burler-cluster-fan.md` and `burler-fix-self-check.md` to allow read-only git, and adds a doc.go paragraph saying so.
The older "Fork discipline" paragraph in the same file still lists "no git commands of any kind" as the boilerplate's content.
The board entry asks for "any doc text resting on that premise" to be updated, so this is a missed site.

## Checked and held

- Config loading: the six keys are declared in the loom and darn templates, set to `CategoryGuard`, listed in `ConfigOpenMaps`, and `darn` is registered with `OpenMaps` in `configreg`.
  Tests refuse an empty list, a wrong shape and an undefined fan name for the loom keys, and for the darn keys, naming module, key and entry.
- The move of `FanList`, `FanShapeError` and fan validation into `burlerengine`, and `KeyAtLine` into `configengine`, leaves no reference to the old names.
  The Discussion and Plan keys go through the same `ValidateFanKeys`, with the same messages.
- Wiring: the `RowClusterFans`, `RowFixFans` and `RowModelKeys` entries are correct per row.
  Merge and Darn take Webster's model keys, and the Darn fixer key reads `review_fix_fan in darn.yaml`.
  `wireDarnFans` adds the review row's fixer list, so a darn row records fans exactly when its review row does.
- Fix-half recorded-fan requirement: `RequiresFans` follows `keys.FixFan != ""`, which matches the summary's `goto` way forward.
- Stencils: the read-only git allowance replaces the ban in both forks' boilerplate and keeps every other rule in those sentences.
  The lens texts name rubric headings that the Webster and Merge rubrics carry, and qualify the rest with "whichever this round's rubric carries".
- Test economy: the added rows extend the existing table tests, and no new assertion pins a stencil's static prose.
- The summary's claims hold against the diff, including its "Check by hand" caveats about the audit's subcommand list.

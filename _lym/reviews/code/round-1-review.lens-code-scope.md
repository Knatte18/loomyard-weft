```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (the board entry's `## Goal`; the record has no `Added after Discussion` entries):
- G1: leave the arras keys out of the seeded profiles. Built by C1.
- G2: leave the arras keys out of the `lym board profiles` listing. Built by C1: the listing reads the loaded profiles, as the `stepProfiles` assertions show. C3 pins it.
- G3: mark arras in `docs/overview.md` as kept in code but not offered. Built by C5.
- G4: mark arras in `docs/reference/weaving-glossary.md` as kept in code but not offered. Built by C6.

**Changes:**
- C1: `internal/runprofile/template.yaml` drops `defaults.arras` and the `arras` block from `speed`, `light`, `solid` and `wide`. Traces to G1 and G2.
- C2: `internal/runprofile/runprofile_test.go` drops arras from the expected defaults in `TestLoad`. `TestTemplateLadder` now asserts no arras block and no arras default. Forced by C1: these tests pin the template.
- C3: `internal/boardcli/verbs_test.go` drops arras from the expected defaults in `stepProfiles`. Forced by C1, and pins G2.
- C4: `internal/runprofile/doc.go` gains a sentence saying no starting profile or default covers arras, so an arras run naming no profile runs on the hub's own config. Forced by C1: this doc describes the template's starting profiles and would otherwise leave the arras rule unstated.
- C5: `docs/overview.md` changes the `internal/loomrecipe/` and `internal/shedcli/` tree lines, the `lym loom` served-recipes sentence and the batten `Run-Shed` sentence, each marking arras as kept in code but not offered. Traces to G3. The added line "No starting profile or default covers `arras`…" in the runprofile module paragraph is forced by C1, and it marks arras as G3 asks.
- C6: the glossary's arras row now says "kept in code but not offered", and adds that no starting run profile covers it. The first part traces to G4, and the added clause is forced by C1.

**Findings:** none. Every change traces to a goal point or is forced by C1, and every goal point has a change that builds it. No commit in the range carries a `Pattern-Departure:` trailer, and no change needs one.

Tool calls: none.

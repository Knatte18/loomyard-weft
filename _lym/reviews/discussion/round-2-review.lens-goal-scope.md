```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. I checked the decision record against the `validator-checks` board entry and any parent decision notes.

**Goal points of the entry and whether the record meets them**

- **Citation check (every repository path and Go symbol cited exists):** met by D3 and D4. Two narrowings are recorded: Scope Out excludes unqualified names, and Auto-mode assumptions says the check runs on loom records only.
- **Acceptance check (every decision id appears in an acceptance criterion):** met by D1 and D2. They run on loom and darn records.
- **"the gates of the rows that run it" (discussion side):** met. D2 runs through `ValidateLoomDiscussion` and `ValidateDarnDiscussion`, so both recipes' gates and the verb run it. The narrowing of D3 and D4 to loom is recorded.
- **A card adding a field to a shared `Deps`/`Env` struct lists its reflective seam tests:** met by D5 and D6. The record widens this to every reflected struct and to any target that names the struct. Both widenings are stated and listed under Auto-mode assumptions.
- **A card adding a test file states exactly one tier tag:** met by D7. D7 also accepts a `||` disjunction and `untagged`. The D7 text says why, but Auto-mode assumptions does not list it (NIT below).
- **A card whose Intent changes a package's behaviour lists the package's `doc.go`:** met by D8. The proxy reading is listed under Auto-mode assumptions.
- **Each refusal names the item and what the writer adds:** met. Every new finding in D1–D8 names its item and gives a way forward, and the Constraints section cites `PATTERN-refusal-way-forward`.
- **Writer stencils state the new rules:** met by D10 in the form `PATTERN-stencil-ownership` allows: the rules go in the specs and the stencils get pointer sentences. The Constraints section records this conflict and how it was settled.
- **Entry Out of scope (lens/profile changes; design-interaction and unbounded-edge checks):** respected. The record's Scope Out lists both.
- **Unlicensed work or deferral:** none found. The extra Scope Out items (the report's other check ideas, the relevant list's paths) defer work the entry never asked for. The `//lym:seam` marker, scan and PATTERN-test-speed change are infrastructure that the seam-test goal point needs.

**Decision notes read**

- `lym board list` shows no note titled `Decision for validator-checks:`, so I read no decision notes and none were out of reach.
- The record has no `Added after Discussion` entries.

**Findings**

1. **NIT · consistency · decision-record.md `## Auto-mode assumptions` (D7)**
   - **Summary:** D7 relaxes the entry's "exactly one tier tag" to also accept a `||` disjunction and `untagged`. Every other reading that differs from the entry is listed under Auto-mode assumptions, but this one is not.
   - **Evidence:** decision-record.md lines 128 and 186–191 (the D7 text and the assumption list); the board entry body says "states exactly one tier tag for it".
   - **Fix:** add a one-line assumption naming the relaxation.

**Tool calls**

- `lym board list`, filtered: to find notes titled `Decision for validator-checks:` and the neighbouring entries. My context had only this task's body, not the board list.

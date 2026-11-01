```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set found:**

1. **`PATTERN.md` new `## Code` section (three entries).**
   - Entry format matches the existing entries: backticked ID, an em dash, a rule, then an optional `— [background](…)` link.
   - `PATTERN-no-hard-coded-choice` links to a background file that exists in the diff.
   - `PATTERN-doc-go-documentation` and `PATTERN-named-constants` have no background file, which the record's "where the one line needs one" allows.
   - Code that reads `PATTERN-` ids and sections exists: `burlercli/run.go`, `discussionparser/patternchanges.go`, `planparser/sections.go` and `patternlint`. None of it hard-codes the section names, so the new section needs no code change. Covered. I did not open those files; this rests on a grep of file names only.

2. **`PATTERN-fabric-isolation` entry and background file.**
   - Under "Fabric and git", the entry sits next to `PATTERN-fabric-git` and `PATTERN-heddle-instructions`.
   - The background file says "Fabric is `internal/fabricengine` and `internal/fabriccli`". Both directories exist.
   - It claims fabric already imports board, IDE, heddle, reed and webster packages as debt. Non-test imports in those two directories confirm `boardengine`, `vscode`, `heddleengine`, `reedengine` and `websterengine`.
   - The background file says nothing false about shuttle. Fabric has no shuttle import, and the debt list leaves shuttle out. The focus question about a shuttle provider seam concerns `PATTERN-shuttle-provider-seam`, which already exists. Covered.
   - Consistency with `PATTERN-heddle-instructions`: that entry says "Fabric hands heddle paths and events and names no heddle file". The new entry's "gains no import … no knowledge of their files" is consistent, not contradictory.

3. **`PATTERN-no-hard-coded-choice` background file.**
   - It cites the plan parser reading an absent language as `go`, in the summary's debt claim. `internal/planparser/parse.go:146` (`language := "go"`) and `plan.go:55` confirm this.
   - It is accurate and described as debt.

4. **`PATTERN-shed-shared-machinery`.** It is placed under "Shed and loom" and has no background file or `(test)`. It is consistent with `PATTERN-shed-driver`, which names specific owner packages, and does not contradict it. Covered.

5. **`PATTERN-lym-instructions-in-code`.**
   - It sits under "Agents and prompts" after `PATTERN-heddle-instructions`.
   - It governs what goes into heddle, while `PATTERN-heddle-instructions` governs how heddle's store is rendered.
   - It does not duplicate `PATTERN-stencil-ownership`, which is about reading stencils at call time and not embedding them. Its "stencils and specs" wording is consistent with that entry.
   - It has no cross-reference from `PATTERN-heddle-instructions`. None is required. Covered.

6. **Declared PATTERN changes landed.** The focus file declares no changes list, and the six board-entry invariants each land as exactly one entry. Nothing is raised.

7. **Commit message and departures.**
   - No `Pattern-Departure:` trailer appears, and none is needed because no entry forbids the diff's way.
   - The message claims only the six entries, and the diff bears that out.
   - The commit trailer reads `Co-Authored-By: Claude Opus 5.5`, which differs from this session's attribution reminder. I treat that as outside the affected-set lens and raise no finding.
   - `summary.md` says "the overview grows by about 1.1 KB … `lym loom lint-pattern` passes". I did not re-run the lint (mechanical gate, not for this review).

**Findings:** none.

**Tool calls (3):**
1. Listed the non-test imports of `fabricengine` and `fabriccli`, `"go"` literals, and PATTERN-related files. My context lacked the import list needed to check the debt claim.
2. Listed the non-test files that reference `PATTERN-`, and the files that reference `PATTERN-fabric-isolation` or `PATTERN-heddle-instructions`. I needed to know whether code parses section names or pins entries.
3. Grepped `"go"` in `planparser/parse.go` and `plan.go`, to confirm the absent-language default claim.

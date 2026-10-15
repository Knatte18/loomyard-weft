---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/webster/webster-template-master.md:33
    summary: the Orientation paragraph still tells Master to read every state file it touches as an ordinary file, which sits beside the new rule that Master reads only three kinds of file
    fix: In webster-template-master.md line 33, change "every other state file you touch is named by an explicit path in this prompt or a verb's envelope. Read and write them all as ordinary files." so it no longer licenses reads beyond the new read set, for example "every other state file you touch is named by an explicit path in this prompt or a verb's envelope; write them as ordinary files, and read one only where this prompt tells you to." Then update the pin in contracts/stencils/claims_test.go that asserts "Read and write them all as ordinary files" in the webster-template-master.md row to the new phrase, and keep "{{.plan_dir}}` holds the plan" as it is.
  - id: F2
    severity: NIT
    class: consistency
    location: internal/websterengine/doc.go:9
    summary: the sentence that each fork reads its cards' code itself sits after the sentence about what the package provides, detached from the opening sentence that card 1 says carries it
    fix: In internal/websterengine/doc.go, delete the added line "// Each fork reads the code its cards need itself." and put the clause into the opening sentence instead, after "then forks one implementer per execution batch in-session (Claude Code's Agent tool, subagent_type "fork")", for example ", each fork reading the code its cards need itself,". Keep one sentence per comment line and run `lyx loom lint-comments internal/websterengine/doc.go`.
---

### [MEDIUM] Orientation paragraph still licenses reading every state file

**Location:** contracts/stencils/webster/webster-template-master.md:33
**Issue:** Card 1 narrows Master's reads to `00-overview.md`, the `pattern/` files and a named findings file, and adds the forbid "NEVER read, grep or quarry any other file" at line 219.
The paragraph two lines below the Orientation reads, which the card leaves untouched, still says "every other state file you touch … Read and write them all as ordinary files."
A Master reading the two together has no clear answer on whether it may open a state file such as the batch report or `state.json`.
The forbid is aimed at code, cards and docs, so the likely intent is that state files stay readable where the prompt says so, but the template never says it.
The rendered-prompt tests do not catch this, because they pin the new phrases and never the old line.

### [NIT] Doc sentence placed away from the opening sentence

**Location:** internal/websterengine/doc.go:9
**Issue:** The package doc's opening sentence now says Master reads the plan's overview and the repo's rules once, never the code, then forks one implementer per batch.
The clause that each fork reads its cards' code itself landed as a separate line after "provides only those verbs' logic plus the distillation behind them.", in the middle of a paragraph about the package's scope.
Card 1 places this statement in the opening sentence.

## Checked, no finding

- Master template: the Orientation names only the overview and the conditioned `pattern/` read, and no other sentence names `pattern/` without the condition.
  No sentence tells Master to read cards, sources, tests or docs, bar the line-33 case in F1.
- FRESH-READ fragments: the plain, pack and pack-card fragments carry the same two statement lines.
  `webster-fragment-fresh-read-pack-inline.md` is absent from the diff, so it is byte-identical, and `inlineFreshReadBody` is unchanged.
- The fork prefix, the recovery prefix and the verify-fix body orient from the overview and the rules and send the reader to the code per card or per finding, consistent with the Master wording.
- Pins: every new claim extends an existing row of `wordingClaims`, and every render pin extends an existing `TestRenderPrompts` row.
  No new top-level test and no stencil marker added or removed, and each pin is a short phrase.
  `lyx gate test ./contracts/stencils/... ./internal/websterengine/...` passes.
- The diff changes no Go behavior beyond a doc comment, test constants and test rows.
- `docs/overview.md` and `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` match the new behavior, and `grep -rn "reads the codebase"` over the three docs finds nothing.

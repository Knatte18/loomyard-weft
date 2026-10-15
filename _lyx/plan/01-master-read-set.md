# Card 1 — Master reads the plan overview and the rules, never the code

**Edit:**
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`

**Intent:** Narrow webster's Master to the read set of decision D1, and say so in the three docs (D6).

In `webster-template-master.md`:

- The banner comment's "the long-lived session that reads the codebase and the plan once" becomes "the long-lived session that reads the plan's overview and the repo's rules once, never the code".
  The rest of the banner is unchanged.
- The opening paragraph under the two `>` blocks drops "you read the codebase and the plan once, up front," and "inherits everything you have already read — no cold orientation, no codebase tour, per batch."
  It says instead: you stay alive for the whole plan, you read the plan's overview and the repo's rules once, up front, and every implementer you spawn is an in-session fork that inherits those reads;
  you never read the code, since forks rewrite it and a code read a later fork inherited would be stale, and each fork reads the code its own cards need itself.
  The sentence "You never edit code yourself, you never run git, and you never use a `/model` switch." stays.
- `## Orientation — read this ONCE, up front` keeps its heading (the `template_test.go` order check against the PATTERN directive pins it).
  Its first sentence drops "read the codebase's structure and conventions, and" and keeps the read of `{{.plan_dir}}/00-overview.md` and its section list.
  A new sentence adds: when a constraints section stands above this one, also read the `pattern/` background files it points at, choosing by what the forks will do as `00-overview.md`'s Card Index and `## Shared Decisions` describe it.
  This sentence must contain the exact phrase "the `pattern/` background files the constraints section above points at, when that section is present", so the read reads as vacuous when the pattern directive renders empty; no other sentence of the template names `pattern/` without that condition.
  The existing "You do NOT pre-read every card's own `NN-<slug>.md` file" sentence stays.
  "This is the stable context every fork you spawn inherits instead of re-deriving it cold each time." becomes a sentence saying these reads are what every fork inherits, and they stay current for the whole run because the run refuses a drifted plan.
  The template stays free of `{{if}}`/`{{range}}` and keeps every marker it has.
- In `## What you never do`, "You read files with Read and Grep and write your two contract files with Write." becomes "You read only `{{.plan_dir}}/00-overview.md`, the `pattern/` background files the constraints section above points at, when that section is present, and the findings file a `Gate findings recorded at …` message names, and you write your two contract files with Write."
  A new line follows it: "NEVER read, grep or quarry any other file — no card file, and no source, test or doc file: forks rewrite the code, so a read a later fork inherits is stale."

In `contracts/stencils#wordingClaims`, the `webster-template-master.md` row:

- The claim group "Master needs no script to read or write a file: it reads with Read and Grep …" changes its phrase to "You read only `{{.plan_dir}}/00-overview.md`" and "you write your two contract files with Write", with a why saying Master's reads are limited to the overview, the rules and a named findings file, and it writes its two contract files with Write.
- A new `wantAll` group pins the read set and the forbid: "read `{{.plan_dir}}/00-overview.md` once", "the `pattern/` background files the constraints section above points at, when that section is present", "NEVER read, grep or quarry any other file", "forks rewrite the code, so a read a later fork inherits is stale".
- A new `wantNone` group bars the retired framing: "read the codebase's structure and conventions", "inherits everything you have already read", "codebase tour", "Read and Grep".

In `internal/websterengine_test#TestRenderPrompts`, the row "master with PATTERN inactive renders no constraints block" (a render with an empty `pattern_directive`) also asserts `contains` "/00-overview.md` once", the `## Orientation` sentence's own read, and `notContains` "codebase".
The row's render func also counts, in the rendered text, the occurrences of "`pattern/`" and of "the `pattern/` background files the constraints section above points at, when that section is present", and calls `t.Errorf` when the two counts differ, so a `pattern/` mention outside the conditioned phrase fails (acceptance criterion 3).
That row fails if the rendered empty-directive prompt drops the Orientation read of the overview, names `pattern/` without the condition, or brings back a codebase read; the forbid line is pinned by the claim row alone.
The row "master places the PATTERN directive ahead of the first work instruction" gains "when that section is present" in its `inOrder` list after "## Orientation", so the PATTERN-active render carries the conditioned read after the constraints section.

`internal/websterengine/doc.go`'s package opening sentence says one long-lived Master session reads the plan's overview and the repo's rules once, never the code, then forks one implementer per execution batch in-session, and each fork reads its cards' code itself.
Every edited comment line holds one sentence (`PATTERN-comment-line-breaks`); run `lyx loom lint-comments` on the file.

`docs/overview.md`'s webster module line and the opening paragraph of `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` replace "reads the codebase and the whole plan once" (in the suite doc, "reads the codebase and the whole flat card-list plan (…) once") with the same statement: the Master reads the plan and the repo's rules once, never the code, and each fork reads its cards' code itself.
The suite doc keeps its parenthetical about the plan format and the batchifier.

**ImpactSummary:** One stencil, its claim row and one render row, plus three docs; no Go behavior, marker or caller changes, and the `## Orientation` heading other tests pin stays.

**Commit:** 1: webster: Master reads the plan overview and the rules, never the code

**Verify:** `! grep -n "reads the codebase" internal/websterengine/doc.go docs/overview.md tools/sandbox/SANDBOX-WEBSTER-SUITE.md`

# Card 5 — The alignment agent settles landing order itself, and both stencils name the board payload

**Edit:**
- `contracts/stencils/loom/loom-template-alignment.md`
- `contracts/stencils/loom/loom-template-arbiter.md`
- `contracts/stencils#wordingClaims`
- `internal/alignment/doc.go`

**Intent:**
D3 and the stencil half of D5.

D3: `loom-template-arbiter.md`'s read list gives the board entry as `` `lyx board get '{"slug":"{{.slug}}"}'` ``, the JSON payload form the verb takes.
`loom-template-alignment.md`'s two board lines (under `parent-review` and `pr-gate`) give `lyx board list`, then `` `lyx board get '{"slug":"{{.slug}}"}'` `` for this task and the same payload with each slug for every entry that touches it.

D5, in `loom-template-alignment.md`, which stays a stencil-store file read at call time (`PATTERN-stencil-ownership`) and restates no recipe row (`PATTERN-producer-pointer-rule`):
- The `## When to escalate` section is rewritten.
  An overlap with a peer run or a landed parent commit no longer escalates on its own.
  The agent escalates on an overlap only when the designs conflict: both runs change the same behaviour in different ways, a recorded decision in one contradicts or duplicates one in the other, or both change the same PATTERN entry in different ways.
  A landed parent commit holds no decision record, so the agent compares the commit's subject and diff against this run's decision record and diff, and escalates when the commit changes the same behaviour or the same PATTERN entry differently than this run's decisions do, or already makes one of this run's decisions.
  A text-only overlap, the same file or package changed at different places, is never a reason to escalate.
  An open question that needs a decision the board and the records do not give still escalates, and the shared-bookkeeping-file sentence stays as it is.
- A new `## Landing order` section: for every overlap the agent applies the rule that the run furthest along by its recipe row lands first and the other takes its changes at Merge-In, comparing each peer's `row` from `lyx loom peers` with the list's `own_row`, the order the running binary's embedded recipe gives; it never reads a recipe file.
  Two runs on the same row are a tie, and a peer whose `row` is 0 is an unknown order.
  The verdict is unchanged by the order.
- `## Otherwise` and `## What you write` say the report records each overlapping peer or parent commit, what overlaps and the landing order the rule gives, or the tie or unknown order.
- The rule applies at both points, and the read lists keep `lyx loom peers` and `peers show`.

Bound: the agent gains no action; it only stops escalating text-only overlaps, and the parent lock and Merge-In already make whichever run lands second take the first's changes.
A text-only overlap that interacts in behaviour is caught only by the verify at the second run's Merge-In, at `parent-review` and at `pr-gate`, where an approve lands the run with no orch turn.

`wordingClaims` pins it in `contracts/stencils/claims_test.go`:
- the arbiter entry gains `wantAll` of `` `lyx board get '{"slug":"{{.slug}}"}'` `` and a `wantNone` of `lyx board get {{.slug}}`;
- the alignment entry keeps its existing `wantAll` and `wantNone` groups and gains a `wantAll` of `` `lyx board get '{"slug":"{{.slug}}"}'` ``, "`row`", "`own_row`", "lands first", "design conflict", "text-only overlap", "never reads a recipe file", "tie", "unknown order", "subject and diff" and "already makes one of this run's decisions", and a `wantNone` of "Escalate on a substantive overlap" and "a Go package or code file that this run and another run", so the overlap-escalates rule cannot return.

`internal/alignment/doc.go`'s opening paragraph says the agent escalates on a design conflict or a question it cannot settle, and records the landing order of every overlap instead of escalating it; the verdict file's keys and `ReadVerdict` are unchanged.

**ImpactSummary:** Two stencil-store prompts, their pinned phrases and one package doc paragraph; no Go behaviour changes, and the verdict file keeps its keys.

**Commit:** 5: Settle landing order in the alignment stencil and name the board payload

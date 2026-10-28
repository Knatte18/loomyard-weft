# Machine checks in validate-discussion and validate-plan for reviews

`lym loom validate-discussion`, `lym loom validate-plan` and the gates that call the same functions now refuse the recurring review findings a machine can see, before a reviewer spends a round on them.
The themes come from `docs/research/review-setup-audit-2026-10-28/report.md` section 3: wrong code claims and uncovered decisions in the decision record, missing seam tests, tier statements and `doc.go` listings in plan cards.
Each finding names the item it found and what the writer adds to fix it.
The rules live in `loom-discussion-spec.md` and `loom-plan-spec.md`; the Discussion and Plan writer stencils gain one pointer sentence each and restate nothing (`PATTERN-stencil-ownership`).

## Decision record checks

- **Decision ids and acceptance** (both loom and darn records): every writer H3 under `## Decisions` is `### D<n> — <title>` with unique ids (`decision-heading-malformed`, `decision-id-duplicate`), and every id appears as a whole word in an acceptance bullet (`decision-not-in-acceptance`).
  They run beside `discussionparser.Validate`, not inside it, from `loomshed.ValidateLoomDiscussion` and `loomshed.ValidateDarnDiscussion`, so one re-prompt carries every defect.
  `lym loom decision add` still runs `Validate` alone, so a run whose record predates ids is not refused.
- **Cited paths** (loom only): a backticked token that reads as a repository path must exist in the worktree (`citation-path-missing`).
- **Cited symbols** (loom only): a glyph or a `pkg.Member` token must resolve through quarry (`citation-symbol-missing`, `citation-symbol-ambiguous`).
  `planindex.Index` gains a method that resolves a list of glyphs against a worktree root, implemented in `internal/planglyph`.
- A token marked ` (new)` or ` (removed)` after its closing backtick is not tested.
  Fenced blocks, `Added after Discussion` entries, suffixed `## PATTERN changes` bullets and carry-over blocks are skipped.
- `internal/discussionparser` extracts the citations and stays stdlib-only; `internal/loomshed` tests them.

## Plan checks

All three are blocking context checks in the plan gate, beside `rename-tool-missing`.
They run at the Plan-Write and Plan-Burler-Fix gates and in `validate-plan`'s no-flag, `--rework`, `--lens-review` and `--overlay` modes.
They skip cards the run record holds done, and plans whose `language:` is not `go`.

- **`seam-test-unlisted`**: a card whose targets name a struct marked `//lym:seam`, a member of it, its declaring file or its package self glyph must list each of the struct's seam tests, or a later card must edit it.
- **`test-tier-unstated`**: a created `_test.go` file needs Intent lines stating exactly one tier tag (collected as a set), tags joined by `||`, or `untagged`.
  Tier tags are the `gate.yaml` tier keys; with no `gate.yaml` the check is skipped.
- **`doc-go-unlisted`**: a card touching a package's non-test Go code must list that package's `doc.go`, target the package self glyph, or leave the `doc.go` to a later card's Edit or Extends.

## The `//lym:seam` marker

A test that reflects over a struct's fields carries `//lym:seam <unit>#<Type>`, one line per struct.
The marker is parsed in `internal/impactset` beside `//lym:guard`, and the two stack in either order with `//testtiming:keep`.
A tier0 scan refuses a reflective test without a marker, a marker that names a non-struct, and a test reaching `envkit.NilSeams` whose markers miss a struct `Env` nests.
Today's reflective tests are marked.
`pattern/PATTERN-test-speed.md` documents the marker.

## Design decisions to know

- A quarry outage in the Discussion-Write and Discussion-Burler-Fix gates now halts them as an infrastructure fault (`planindex.ErrQuarryUnavailable`), where they passed before.
- The citation check is loom-only: a darn record is checked at handoff, after the code, so existence would test the code and not the claim.
- Seam markers cover every reflected struct, not only `Deps` and `Env`, and `NilSeams` tests mark the whole nested set `Env` holds.
  A plan changing the wiring's dependency structs therefore takes more seam-test listings than one changing `Env` alone.
- `pkg.Member.Sub` is tested by its `pkg.Member` prefix only, since quarry gives a struct field no glyph.
  Lower-case members, packages outside the worktree and unqualified names are untested.
- A `(new)` mark or a `<placeholder>` part exempts a token, so a writer can exempt a wrong claim; review still reads it.
- A `Uses` listing satisfies the seam and `doc.go` checks even when the card falsifies the test or doc; review and the card gate's test run catch that.
- One `Pattern-Departure:` is recorded: the citation integration test builds its repo with `git init` through `gitkit.Git`, since `gitkit.CopyRepo` is pinned to `internal/lymcwd` by `PATTERN-leaf-packages`.

## To check by hand

- `test-tier-unstated`'s way-forward text (`internal/loomshed/testtier.go`): confirm it reads consistently with the plan spec's tier rule.
- A run in Discussion or Plan review when this lands meets the new checks at its next gate and takes one re-prompt.
- `landing-settles-in-place` changes `internal/loomcli/landingdeps_test.go`, which now carries a marker; a reflective test another landing adds before this one needs its marker too.
- The tier tag check assumes each `gate.yaml` tier key is its build tag; on a hub whose tier commands pass other `-tags` values the check demands the key as the tag.
- The plan-level gate run failed once in `github.com/Knatte18/loomyard/cmd/lym`; the fix commit `06e6a516` brings `validate-plan`'s `Long` help page under the byte cap and gofmts `internal/planglyph/deleteorder_test.go`.
  Re-run `lym gate test --upto 2` on `main` after landing.

# Decision record — commit-signature-credit

## Goal

A landing commit and its pull request carry no `Co-Authored-By` line and no `Lym-Task` line.
They end instead with one plain-line credit block, on by default, naming LoomYard and each engine that ran a session in the task.

## Scope

- `internal/landingshed`: the config keys, the block's composition, Finalize's landing commit and pull-request merge message, and PR-Review's pull-request body.
- `internal/landingshed/template.yaml` and the landing config's reconcile migration.
- `internal/summaryparser`'s Co-Authored-By finding detail, which names Go appending a trailer.
- Docs the change makes stale: `internal/landingshed/doc.go`, `docs/overview.md`, `contracts/specs/loom-final-summary-spec.md` and `contracts/stencils/landing/landing-template-describe.md`.
- The test seam list in `internal/testkit/envkit/envkit.go`, which names the retired config field.

Out of scope: a Codex engine; `fabricengine`'s `Warp-SHA` trailer, which is weft bookkeeping and never lands in the code repository.

## Decisions

### D1 — Four new landing.yaml keys replace trailer_text and engine_trailers

`credit` (bool, default `true`) turns the block on or off.
`credit_heading` (default `Created using`), `credit_loomyard` (default `LoomYard https://github.com/Knatte18/loomyard`) and the open map `credit_engines` (default `claude: "Claude <noreply@anthropic.com>"`) hold one line's text each.
All four are cross-pair keys, since they shape what lands on the parent.
`credit_engines` keeps the engine-name check `engine_trailers` had: a key naming no engine fails the load.
Rationale: the board entry asks for each line as its own config text and calls the block optional; a bool states off plainly, where emptying texts would be an implicit switch.

### D2 — The block's shape

With `credit` on, the block is the trimmed heading, then ` - <text>` for the LoomYard line, then ` - <text>` for each engine in the run's engines-run list, in engine-name order, each once.
An empty or whitespace-only text adds no line; a block with no item lines, or with `credit` off, is empty, and the commit and pull request then carry nothing after the body.
The lines are plain text, written after a blank line, never formatted as git trailers.
Rationale: matches the board entry's example block; the engine detection is the existing engines-run read, unchanged.

### D3 — Lym-Task goes

The `Lym-Task: <slug>` line and its helper are removed; nothing else adds a LoomYard trace to the landing commit.
Rationale: the board entry removes it outright.

### D4 — The pull request's body carries the same block

PR-Review creates and edits the pull request with the change description's body, a blank line and the block, composed by the same function the landing commit and the pull-request merge message use.
PR-Review reads the engines-run list when it sets the body; a read failure is an error, as at Finalize.
Rationale: the board entry asks for one signature on the commit and its pull request.

### D5 — Retired keys are dropped, never migrated

`trailer_text`, `engine_trailers` and `co_authored_by` become retired keys: a hub file still holding them loads, and reconcile prunes them.
The `co_authored_by` to `engine_trailers` migration is removed.
Rationale: their values are the co-author lines the operator no longer wants; carrying them into `credit_engines` would keep the text the task removes.

### D6 — The Codex line waits for the Codex engine

The template carries the Claude engine line only, since `credit_engines` refuses an engine name `modelspec` does not know, and `codex` is not one yet.
The Codex engine's task adds `codex: "Codex <codex@openai.com>"` to the template when it adds the engine.
Bound: no Codex session can run before then, so no landing misses its line.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads the layers; retired keys go through the spec's `RetiredKeys`.
- `PATTERN-sparse-hub-base`: no hub base file is filled from the template.
- `PATTERN-test-speed`: tests through `lym gate test`, `t.Parallel`, rows in existing tables over new tests.
- `PATTERN-tolerant-records`: unaffected; no record format changes.

## Auto-mode assumptions

- The heading and item lines render as the board entry shows: the heading flush left, each item line opening with one space, a hyphen and a space.
- The PR body is set on every PR-Review entry that edits it, so a fixer session added after the first entry shows on the next edit, and the merge message at Finalize reads the list afresh.

## Open risks

- An existing pull request opened before the change keeps its old body until PR-Review next edits it.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - prreview-empty-credit-untested (class design, severity MEDIUM, open in rounds 1)
  - prreview-engines-run-failure-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: the template loads with `credit` true and the three texts; a `credit_engines` key naming no engine fails the load naming the key.
- D2: the block is heading, LoomYard line, then the ran engines' lines in name order, each once; an empty text adds no line; `credit` off or no item line yields an empty block and a message ending in the body.
- D3: no landing commit message holds `Lym-Task` or `Co-Authored-By`.
- D4: PR-Review's created and edited body is the description body, a blank line and the block; the pull-request merge message carries the same block.
- D5: a landing.yaml holding `trailer_text`, `engine_trailers` or `co_authored_by` loads, and the migration rewrites none of them into another key.
- D6: the template carries no `codex` entry, and the template loads.

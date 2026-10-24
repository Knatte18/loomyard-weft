## Goal

Write `docs/research/stencil-engine-audit-2026-10-24/report.md`: an audit of every Claude-specific mechanic in the stencils, specs and the Go code that inlines prompt text, with what Codex needs at each site and a size estimate for the rewrite.
Nothing is built.

## Scope

- In: one new file, `docs/research/stencil-engine-audit-2026-10-24/report.md`.
- Read-only inputs: `contracts/stencils/`, `contracts/specs/`, the Go code that renders or inlines prompt text (`internal/loomengine`, `internal/shuttleengine` including `claudeengine`, `internal/orchengine`, `internal/burlerengine`, `internal/websterengine`), and `docs/research/codex-spike-2026-10-24/report.md` for the Codex side.
- Out: any change to a stencil, spec, Go file or the board entry.

## Decisions

- **Sites are listed per mechanic, each with file and line.**
  Rationale: the operator's decision groups sites into named include blocks, so the grouping key is the mechanic.
- **The report proposes one include block per engine-specific mechanic, with the Claude text it would hold.**
  Rationale: this is the operator's decision of 2026-10-24; a stencil keeps one text instead of `{{if}}` branches.
  The existing include mechanism (`stencil.FillWith`, `{{template "name"}}`) is the carrier.
- **A site with no Codex counterpart is marked as such, never given an invented equivalent.**
  Rationale: the Codex column cites only what the spike report observed; anything the spike did not test is marked untested.
- **Specs and Go-inlined text are audited alongside stencils.**
  Rationale: the board entry names both; a Claude word in a Go string reaches the prompt as surely as one in a stencil.
- **Line numbers are the numbers at the audit's commit and are not promised to stay current.**
  Rationale: the report is a measurement of one tree.

## Constraints

- `PATTERN-shuttle-provider-seam`: provider specifics live under `internal/shuttleengine/<engine>`; the block proposal must not move provider words into a shared package.
- `PATTERN-stencil-ownership`: prompt text is read at call time through `internal/stencilstore`, never embedded; a proposed block is a stencil file.
- Markdown uses semantic line breaks; no link needs to resolve outside the new folder.

## Auto-mode assumptions

- No operator is present; the grouping into blocks and the block names are the writer's best judgment.
- The audit is by reading and grep; no site is verified by running a Codex session.

## Open risks

- A site missed by the grep patterns: the audit searches for tool names, fork and message mechanics, skill and settings words, and reads the heaviest files in full, so a purely implicit assumption (for example a prompt that relies on a Claude-only tool behaviour never named) can be absent from the list.
- The Codex column rests on one spike with a cheap model; behaviour of the subagent `spawn_agent` path under real prompts is untested.

## Acceptance criteria

- The report exists at the named path and changes nothing else.
- Every listed site names a file and line, a mechanic, and a Codex need or a "no counterpart" mark.
- Sites are grouped into proposed blocks, each with a name and the Claude text it would hold.
- The report ends with block, site and stencil counts.

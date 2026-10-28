## Goal

Report where agent turns go per role, which serial turns could have been merged, and which stencil or system-prompt wording is most likely to cut turns.
The report is the baseline for later turn-cutting changes; it proposes no handed-over or pre-read source.

## Scope

- Output: `docs/research/turn-audit-2026-10-28/report.md` with the sample size beside every number, and the scripts and TSV tables that produced it beside it.
- Sessions: Claude Code transcripts under `~/.claude/projects/` of loom and darn runs that landed since 2026-10-20, found by the worktree-named project directories, forks included; runs from OSL-1046 are left out.
- Transcripts are the only source; the scripts use `bash`, `jq` and `awk`.
- No code, stencil, profile or config change; recommendations are marked as recommendations.

## Decisions

### Turn unit

A turn is one assistant API message, the records sharing one `message.id`; its tool calls are the `tool_use` blocks across those records.
Rationale: the cost grows per API round trip, and the transcript splits one message over several records.

### Mergeable run

A mergeable run is two or more consecutive single-call turns of read-only gather calls where no later call names a path or token that appears in the previous call's result.
Rationale: a call whose input is read out of the previous result cannot be merged; the check is a lower bound of the mergeable set, and the class-only count is reported beside it as an upper bound.

### Cost weight

A turn's cost is the `tools/tokencount` weight, input + 1.25 x cache write + 0.1 x cache read + 5 x output.
Rationale: no price table is in the records, and the weight is the repo's one relative measure.

### Role

A session's role is the part after the last colon of its transcript title; a fork counts under its parent's role with `+fork` appended.
Rationale: this is how `tools/tokencount` names roles.

### Sample cut

Only sessions that started on or after 2026-10-20, under a slug with an `archive/<slug>/*` tag created on or after that date, are counted.
Rationale: a slug's earlier runs share its project directory.

## Constraints

- PATTERN: no production code changes, so no PATTERN entry applies beyond the docs-only rules.
- Markdown with one sentence per line.

## Auto-mode assumptions

- Roles with no transcript on this machine yield no row; the report names them as not measured.
- A role with fewer than five sessions is reported with its n and no recommendation estimate stronger than "indicative".

## Open risks

- The sample is the transcripts present on this machine; most loom runs' transcripts are absent, so the loom-only roles rest on very few sessions.
- The dependency check is a text match and can mark a call dependent that was not.

## Acceptance criteria

- Per role: sessions, turns per session, calls per turn, share of single-call turns, each with n.
- Per role: mergeable runs, their count, and their share of session cost, with the class-only upper bound.
- Per role: reads after the first edit, read-edit switches, edits per turn, re-edited files, gate/test/lint calls per session.
- A table of the tools and commands that make up the mergeable turns.
- Per role: a stencil or system-prompt recommendation with an estimated turns and cost saving, marked as a recommendation.
- Every script runs from any worktree and writes the TSV beside it.

# An MCP server over lyx exposing the verbs agents need

`lyx-mcp` · Note · feature

## Why

Agents misuse or bypass lyx (the hub read `_board/*.json` with jq) because the CLI shape is not discoverable.
An MCP tool carries a typed input schema and a description.

## Constraint from the operator (2026-10-01)

Not one tool per verb: MCP entries are heavy.
Only the important ones are exposed, and how to pick them is still open.

## Proposed selection

1. **Measure first.** Mine transcripts and logs for which lyx commands agents run, where they fail, and where they read around lyx.
2. **Few coarse tools.** One tool per area with an `action` enum (the pattern of Claude Code's Artifact tool), e.g. `board`, `run` (status, approve, reject), `webster` (Master only); a handful in total.
3. **Per-role exposure.** A fork may get none, Master gets `webster`, the hub gets `board` and `run`.

## Shape

- A thin server shipped in the ly plugin; each action calls the same module seam the CLI uses, so behaviour never forks.
- The CLI stays for humans and scripts.
- Wins: no shell-quoted JSON payloads, and per-tool permissions (an agent can be denied `accept-audit`).

Related: `bolt-roadmap-tiers`, `process-naming`.


## Status: needs a careful design session first

Operator 2026-10-01: what goes into the MCP must be thought through very carefully; this is not a loom task yet.
Questions for that session:

- Which verbs need an MCP tool at all, and which are served well enough by a better `--help` (payload schema and an example per command) and a per-role command card in each stencil, generated from the cobra tree?
- Is permission gating (an agent denied `accept-audit` or `approve`) the real reason for MCP, since `--help` and cards cannot give it?
- One tool per module was raised and judged too heavy (long descriptions); what is the cut instead?

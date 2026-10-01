Why: agents misuse or bypass lyx (e.g. jq on _board/*.json) because the CLI shape is not discoverable; an MCP tool carries a typed input schema and a description.
Operator 2026-10-01: NOT one tool per verb, MCP entries are heavy; pick only the important ones, and how to pick is open.
Proposed selection: (1) measure first: mine transcripts and logs for which lyx commands agents run, where they fail, and where they read around lyx; (2) few coarse tools, one per area with an `action` enum (the Artifact-tool pattern), e.g. board, run (status/approve/reject), webster (Master only), a handful in total; (3) per-role exposure: a fork may get none, Master gets webster, the hub gets board and run.
Shape: a thin server in the ly plugin, each action calling the same module seam the CLI uses so behaviour never forks; the CLI stays for humans and scripts.
Wins: no shell-quoted JSON payloads, per-tool permissions (deny accept-audit to an agent).
Related: bolt-roadmap-tiers, process-naming.
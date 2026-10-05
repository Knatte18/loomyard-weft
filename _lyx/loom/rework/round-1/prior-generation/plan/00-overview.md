---
format: 5
approved: true
language: go
---

# Plan: Skills and stencils — every role gets its procedure from a stencil and its skills from lyx

Every role lyx spawns gets its whole procedure from a stencil, the orch included, and lyx itself loads that procedure and the skills the role needs on every path into a session.
Shuttle's launch spec names a role's skills and the Claude engine types them before the prompt (cards 1–2).
A shared parent directive is rendered into every spawned role's opening stencil, so roles escalate to their parent rather than the operator (card 3), and each spawning module moves onto both (cards 4–10).
The `ly-drive` skill becomes the driver stencil and leaves the `ly` plugin (cards 10–11).
The orch gets a role stencil and a note in place of `scribe:handoff`, `cycle` and `distill` both write the note first, and the watcher reloads skills, role and note after every clear and compaction (cards 12–15).
Two PATTERN entries with stencil scans close the plan (card 16).

## Card Index

1 — shuttle-skill-loading — the launch spec names skills that shuttle loads before delivering the prompt
2 — claude-skill-loader — Claude types each skill as a slash command and defers the prompt pointer
3 — parent-directive — one shared parent directive package and its stencils
4 — loom-producer-roles — Discussion, Plan and Rework load skills from the spec and render the directive
5 — webster-roles — the Webster master and recovery load skills and render the directive
6 — burler-role — burler rounds load skills and render the directive
7 — bouncer-roles — the Bouncer judge and seed load their skill and render the directive
8 — treadle-roles — treadle targeting, judge and triage load their skill and render the directive
9 — landing-friction-roles — conflict, describe and friction load skills and render the directive
10 — driver-stencil — the loom driver launches from the shed driver stencil
11 — retire-ly-drive-skill — remove the ly-drive skill and reword every mention
12 — orch-role-stencil — the orch role stencil, the note template and the one-line pointers
13 — orch-note-gate — `cycle` clears, `distill` compacts, and both write the note first
14 — compaction-since — shuttle reads whether a session compacted since a given time
15 — orch-reload — the watcher reloads skills, role and note after a clear, a compaction and an auto-compaction
16 — pattern-entries — PATTERN entries for typed skills and the parent directive, with their scans

## Shared Decisions

### Decision: role skill lists live beside the role constant

Each spawning module declares its role's skill list as an unexported value next to the role constant it already owns (`planRole`, `bouncerJudgeRole`, `RoleMaster`, `orchStrandName`, …) and sets it on the spec it builds.
The lists are the decision record's D2 table and nothing else; no language-specific scribe skill is listed for any role.

### Decision: the parent name is told, from the reed geometry's origin record

Every spawning module is told the parent name the way it is told its stencils directory, never deriving it.
The source is always the worktree's origin record, the same one reed injects as `LYX_PARENT`: `hubgeom.ResolveParent` for the webster and burler geometries, and the reed geometry's `ParentName` (kept on the loom receiver as `parentName`) for every role loom wires.
An unresolvable or absent parent is an empty name, which renders the directive's no-parent variant.

### Decision: an unknown skill is detected from the pane, bounded by the timeout

Claude Code's reaction to an unknown `/<skill>` is unverified.
The Claude engine classifies its unknown-command notice from a pane capture; a reaction that ends a turn confirms the load, and any other reaction is skipped at the skill-load timeout, so every case is bounded.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags 'integration smoke' ./...
go test ./...
go test -tags integration ./...

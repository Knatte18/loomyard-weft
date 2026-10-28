# Decision record — claude-md-lym-rules-out

## Goal

This repository's `CLAUDE.md` holds only rules for developing LoomYard itself.
Every rule about how lym behaves in any hub leaves it, and lives in lym's code or in a stencil or skill lym ships, so every hub gets it and no hub reads a LoomYard-only rule as a lym rule.

## Scope

- Sort every rule in `CLAUDE.md` into kind 1 (every use of lym, in any repo) or kind 2 (developing LoomYard here).
- Remove each kind-1 rule from `CLAUDE.md`.
  Where no shipped stencil, skill or code already carries it, add it to the one that reaches the agents that need it.
- Scan the shipped stencils, specs and plugin skills for kind-2 rules that name LoomYard where they should name the hub's own repository, and fix any found.
- Out of scope: new enforcement code, the creel and shuttle work in flight, and any change to this task's own board entry.

## Decisions

### D1 — Rules that leave CLAUDE.md

These are kind 1 and leave `CLAUDE.md`:

- In "PATTERN.md is authoritative": that agents lym spawns get the `PATTERN.md` overview inlined.
  The pattern directive stencils carry it.
- In "Production": that Batten's Land row runs the hub's `deploy` command after a landing into a `deploy_branches` branch.
  `internal/battenshed`'s code enforces it and the orch role stencil tells the orch.
- In "Notes go in git": that task worktrees are torn down with the memory store, that `_lym/raddle/` is regenerated at landing, and where orch-only rules go.
- In "Docs land in the same commit": that an unbuilt design lives in its board entry's body, that the building task never changes its own entry's body or scope, that the landing marks it done and prune removes it, and the run-lock rule.
  The board's run lock enforces the last two, and the discussion, plan, darn and review stencils state them.
- In "Terminology": that the board is read and written only through `lym board`.
  The board skill states it.

Rationale: each describes lym's behaviour or tells an agent how to work with lym in any hub, so it ships with lym.

### D2 — Rules that stay in CLAUDE.md

These are kind 2 and stay: reading `PATTERN.md` before a change and landing a new invariant with its code, the cgo build, `update-plugins.sh` and `./deploy-dev`, plugin versioning, the push rule for `main`, working only in one's own worktree, docs landing in the same commit (`doc.go`, `docs/overview.md`, `PATTERN.md`), interactive tmux over `claude -p`, semantic line breaks, the Merriam, perch, board and Bolt shorthands, and `internal/fslink`.
"Work only in your own worktree" stays because it governs sessions developing this repository by hand, which no lym stencil reaches; spawned roles are confined by their own stencils and settings.
"Notes go in git" stays as a kind-2 rule: a durable note about developing LoomYard goes in `CLAUDE.md` or a code comment, and a rule for every use of lym goes into lym's code or shipped stencils, never into `CLAUDE.md` or a package's `doc.go`.

Rationale: the operator named the cgo build, `update-plugins.sh` and the push rule for `main` as kind 2; the rest are conventions of this codebase's own layout and tooling.
Stating the kind split in `CLAUDE.md` keeps a later edit from putting a kind-1 rule back.

### D3 — Homes for the kind-1 rules no stencil yet carries

- The memory-store rule goes into the orch role stencil's "Scratch and notes" section: a durable note goes into git, never an engine's session memory store, which stays on one machine and goes stale with its worktree.
  The orch is the lym role that keeps notes across sessions; a task role's durable output is its commits.
- The rule that an unbuilt design lives in its board entry's body, and that the building task never rewrites the body to record what was built, goes into the board skill's "Writing a body" section.
- The `_lym/raddle/` regeneration sentence gets no new home: no lym code regenerates `_lym/raddle/`, so it describes no built behaviour and is dropped.

Rationale: every other kind-1 rule D1 lists is already stated by a shipped stencil, skill or enforced by code, so it needs no new text.

### D4 — No stencil names LoomYard where it should name the hub

The shipped stencils, specs and the board skill name LoomYard only for lym's own bugs and friction (`lym selfreport create`, Friction-Reflect) and for the code-review rubric's own-standard line, which are correct in every hub.
No stencil text changes for kind 2.

Rationale: the quarry intake failure is already fixed in the board skill's and orch stencil's intake sections, which take in the hub's own repository's issues.

## Constraints

- `PATTERN-stencil-ownership`: a stencil states its own rule and never restates another producer's format contract.
- Stencil and skill text follow semantic line breaks.
- The stencil tests in `contracts/stencils` keep passing; markers and banners stay unchanged.
- `CLAUDE.md` changes need no test run on its own, but the stencil edits ship with lym and go through `lym gate test`.

## Auto-mode assumptions

- A darn run holds no interview; the kind split above is the writer's own reading of the operator's 2026-10-22 and 2026-10-28 rules in the board entry.
- The orch is the only lym role that needs the memory-store rule.

## Open risks

- A session started by hand in another hub's worktree reads no lym stencil, so the kind-1 rules reach it only through the board skill when it loads it.

## Acceptance criteria

- `CLAUDE.md` carries none of the kind-1 rules D1 lists, and every kind-2 rule D2 lists (D1, D2).
- `CLAUDE.md` states the kind split as its notes rule (D2).
- The orch role stencil's "Scratch and notes" section carries the memory-store rule, and the board skill's "Writing a body" section carries the design-in-body rule (D3).
- No stencil text changes for kind 2, and the stencil tests stay green (D4).

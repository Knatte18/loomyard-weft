---
format: 5
approved: true
language: go
---

# Plan: agent names `<code>:<slug>:<role>` for every strand, with parentage

Every strand lyx spawns through reed gets one hierarchical name — `<code>:<slug>:<role>` in a task worktree, `<code>:<role>` in the prime and in a standalone run — formed by reed under its state lock from a prefix its geometry tells it.
That one name is the strand record's name, the `LYX_STRAND_NAME` env value, the tmux pane title and the Claude session name, and the per-hub watchdog repairs the two display mirrors when they drift.
Each repo gets a short code, given at `lyx fabric clone --code` or later through `lyx fabric code`, and recorded on weft:main as `.lyx-code`; no strand spawns on a hub without one.
Every worktree run records the full name of the session that spawned it in its seed, every strand in that worktree is told it as `LYX_PARENT`, and the ly-drive driver sends its escalations there through SendMessage.
The build order is: the name grammar, the code record and its verb, the shell seam the Claude line needs, the seed's parent, the geometry that tells reed both, reed's name formation and its callers, the env and title mirrors, the Claude flag, the watchdog's repair and its Claude provider, `lyx reed list`, the driver notice, and the docs.

## Card Index

1 — agentname-package — one leaf package owns the name grammar, plus the Agent Name Invariant
2 — fabric-code-record — `.lyx-code` beside `.lyx-warp`, and `lyx fabric clone --code`
3 — fabric-code-verb — `lyx fabric code [<code>]` prints or records the hub's code
4 — shell-envref — `Shell.EnvRef` expands an env var as one argument
5 — seed-parent — the seed's write-once `parent`, filled by batten run, Seed-Child and loom start
6 — geometry-naming — hubgeom and standalonegeom tell reed the code, slug and parent
7 — reed-forms-names — reed forms and numbers full names; `strand_name` retires
8 — callers-address-roles — loom, orch and batten name and find strands by role segment
9 — reed-env-injection — every strand process gets `LYX_STRAND_NAME` and `LYX_PARENT`
10 — pane-title-mirror — the full name becomes the pane title, with `allow-set-title off`
11 — claude-name-flag — Claude's launch and resume lines pass `--name "$LYX_STRAND_NAME"`
12 — watchdog-name-repair — the watchdog repairs drifted titles and session names behind a provider seam
13 — claude-session-namer — the Claude provider for that seam, wired by cliwire
14 — reed-list-verb — `lyx reed list`, the hub-wide name directory
15 — driver-parent-notice — ly-drive and the driver prompt notify `$LYX_PARENT` on escalation
16 — naming-docs — overview, reed, fabric and shedrun package docs

## Shared Decisions

### Decision: the code refusal lives at the spawn, not the geometry

- **Decision:** `hubgeom.ReedGeometry` tells an empty code when `.lyx-code` is absent, and reed's `AddStrand`/`ReplaceStrand` refuse it before booting anything, naming `lyx fabric code <code>`.
- **Rationale:** `reed status`, `down`, `list` and the watchdog daemon must keep working on an uncoded hub; only forming a name needs the code.
- **Applies to:** cards 6, 7

### Decision: a fresh bind is a weft with neither the warp binding nor the anchor marker

- **Decision:** `lyx fabric clone` requires `--code` only when the probed weft carries neither `.lyx-warp` nor `.lyx-anchor`; any weft carrying either is bound, and a bound weft without `.lyx-code` takes `--code` or warns.
- **Rationale:** the decision record's "bound but uncoded" branch exists for repos bound before this task, and a weft holding an anchor was bound by lyx already.
- **Applies to:** card 2

### Decision: a role-less add takes reed's default role `strand`

- **Decision:** an `AddSpec` with neither `Role` nor `NameOverride` is named with reed's own default role `strand`, numbered like any other.
- **Rationale:** `lyx reed add --cmd …` by an operator and reed's own fixtures name no role; refusing them would be a new refusal with no caller asking for it, and `agentname` keeps its role constants to the two addressed roles.
- **Applies to:** card 7

### Decision: name lookups match by role segment through `agentname.Matches`

- **Decision:** every lookup of a strand by name — reed's `--name`, loom's status and driver strands, orch's strand, batten's driver wait — goes through `agentname.Matches`, which accepts the full name, its role segment, or a legacy exact name.
- **Rationale:** one reed state file holds one worktree's strands, so the role segment is unambiguous inside it, and a strand recorded before this task stays reachable by its exact old name.
  The driver is the one role whose literal changes (`loom-driver` to `driver`), so its two lookups also match the legacy literal, or a run in flight across the deploy would miss its own driver.
- **Applies to:** cards 7, 8

### Decision: name repair runs only in the hub watchdog daemon

- **Decision:** `Engine.WatchNames` is a separate loop the daemon's `enterSession` starts beside `Watch`; the standalone CLIs' in-process `Watch` does not run it.
- **Rationale:** the decision record sets a standalone run's mirrors at launch only, and the provider seam is filled in the CLI wiring layer the daemon already passes through.
- **Applies to:** cards 12, 13

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./internal/agentname/... ./internal/fabricengine/... ./internal/fabriccli/... ./internal/hubforge/... ./internal/hubgeom/... ./internal/standalonegeom/... ./internal/shell/... ./internal/shedrun/... ./internal/battencli/... ./internal/loomcli/... ./internal/loomengine/... ./internal/orchcli/... ./internal/reedengine/... ./internal/reedcli/... ./internal/shuttleengine/... ./internal/shuttlecli/... ./internal/burlercli/... ./internal/webstercli/... ./internal/cliwire/... ./internal/treadleengine/... ./internal/burlerengine/... ./cmd/lyx/... ./tools/sandbox/...
go vet -tags smoke ./...

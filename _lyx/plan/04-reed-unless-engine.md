# Card 4 — reed-unless-engine

**Create:**
- `internal/reedengine#Engine.AddStrandUnless`

**Edit:**
- `internal/reedengine/strand.go`
- `internal/reedengine/strand_test.go`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/agentname#Resolve`

**Intent:**
Give the reed engine an add that is skipped while a named strand is live in the worktree (D2), the engine half of #340's fix: the folder-open `reed add claude` stacks a strand below orch, orch collapses to `collapsed_rows`, and its idle probe stalls.

In `strand.go`:

- Add `liveStrandNamed(strands []Strand, name string, aliveIDs map[string]bool) int`, beside `classifyIfAbsent`: the index of the first strand whose `Name` equals `name`, whose `Display.Anchor` is not `render.AnchorHidden`, and whose `PaneID` is non-empty and in `aliveIDs`; -1 otherwise.
  `aliveIDs` is the set `aliveIDSet` builds, never `liveIDSet`, for the reason `classifyIfAbsent`'s doc records: a dead-but-present pane must not count.
- Move `AddStrand`'s body into `AddStrandUnless(spec, unlessName)`; `AddStrand` becomes a wrapper passing `""` and dropping the bool, so its callers and behavior are unchanged.
- In `AddStrandUnless`, a non-empty `unlessName` is resolved to a full name through `agentname.Resolve` with the geometry's shortname and slug, the same rule `--name` uses, before the session pre-flight, so an unformable name refuses without booting a server.
  After `loadOrInitStateLocked`, and before the `IfAbsent` branch, it lists panes and calls `liveStrandNamed`; on a match it returns that live strand and `true`, with no `SaveState`, no reconcile/apply, no launch and no focus change.
  A cold worktree the pre-flight just booted has no live pane, so the add proceeds.

What this lets through, and its bound: the add can now be a successful no-op.
It only ever suppresses an add; it never removes, resizes or refocuses an existing strand, and the name is the caller's, so reed stays role-agnostic.
The check is backend-neutral engine code, so tmux and psmux decide alike.

In `strand_test.go`:

- A table test of `liveStrandNamed` on fixture strand tables, run on every platform: a live visible `orch` matches; a dead pane, an empty `PaneID`, a hidden `orch`, an absent `orch` and a live strand of another name each return -1.
- Engine tests through `newTestEngine` and an exec hook, as the `--if-absent` tests do: a live `orch` makes `AddStrandUnless(spec, "orch")` return the orch strand and `true` with persisted state unchanged and no split-window or layout command issued; a dead `orch` and no `orch` each add the strand and return `false`; an unformable `unlessName` refuses before any tmux command.

In `doc.go`, extend the naming paragraph: an add can name a strand whose live presence skips it, matched by the same rule as an explicit name, and a dead or hidden match does not count.

**ImpactSummary:** `AddStrand` keeps its signature and behavior as a wrapper over the new method; only strand.go, its tests and the package doc change.

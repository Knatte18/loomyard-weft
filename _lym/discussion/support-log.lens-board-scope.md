```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the gate-run-keeps-cache-warm decision record.

**Findings**

1. MEDIUM, `scope`, D4 (and the Goal): the record does not make the cap a config value of its own. Severity is MEDIUM.
   - The entry's brief says "with the cap a config value beside the engine cache lifetime".
   - The body says "The cap is config, not a constant … it can differ per recipe, row or role, with a hub-wide default beside the engine's prompt-cache lifetime (operator, 2026-10-31)".
   - D4 instead derives the cap from the role's `prompt_cache_ttl` less one hub-wide `keep_warm_margin` in `shuttle.yaml`. No cap key sits beside the lifetime in `claude.yaml`.
   - The only way to change one role's cap is to change its cache TTL, which also changes what the cache costs. Two 5-minute roles, such as `darn` and a Webster role on the default, cannot get different caps.
   - So the operator's point that "a darn run may want another cap than Webster" is met only as a side effect of the TTL.
   - Fix: add a cap default and a per-role cap override beside `prompt_cache_ttl` and `prompt_cache_ttl_roles`, keeping the derived value as the default. Or record the departure from the operator's stated idea as a parent question (class `decision`).

2. LOW, `consistency`, Open risks (the `machine-switch` line): the risk is stated loosely.
   - `machine-switch`'s record holds runs through the pause record and does not end processes itself.
   - A detached gate runner ignores pauses. While the agent is held it stops calling back, so D3's lease ends the runner after one cap plus the grace.
   - What is left to settle is whether the switch should wait out or end a live gate runner before it hands work over. The line should say that, or be dropped as covered by D3.

**Goal points of the entry**

- Gate run capped like the slot wait, tiers go on in a process of their own, same command joins and prints the result: met (D1, D2, D3).
- Cap is config and read by the slot wait, the run wait and `lym shuttle await`: met (D4).
- Cap can differ per recipe, row or role, with a hub-wide default beside the engine's prompt-cache lifetime: partly met (finding 1).
- Break-even past which a known-long wait blocks without re-runs, with the expected length from the gate's own records of the key and the break-even as config: met (D5). D5 re-runs once an hour because of the Bash tool's one-hour ceiling, and gives the reason.
- Find the other waits that block past the cache lifetime, checking the driver's `lym shed step` first: met (D8; D6 and D7 add card-verify and the fixer render).

**Entries, peers and parent commits checked**

- `gate-fewer-binaries-trial` (dependency, done, research only): no conflict.
- `machine-switch` (peer, Discussion-Burler-Review): no design conflict; see finding 2.
- `reed-status-buttons`: none.
- `plan-split-producers`: none.
- `batcher-wide-default`: none. Its edit to `internal/websterengine/doc.go` overlaps only as text.
- Notes `run-pins-its-version`, `interactive-idle-park`, `dormant-orch`, `codex-engine`: none. `codex-engine` would answer the lifetime through the same engine seam that D4 uses.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board list shows no `Decision for gate-run-keeps-cache-warm:` note, and the record has no `Added after Discussion` entries. No note was out of reach.

**Tool calls**

- One grep of `machine-switch`'s decision record for gate, process and detach lines. My context held only that peer's status, not its decisions.

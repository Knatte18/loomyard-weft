```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the decision record against the board entry, the touching entries, the peer runs and decision notes. I made no tool calls: my context already held the board list, this task's entry, `lym loom peers` and the touching entries.

**Findings**

1. `MEDIUM` (design), D1/D3, entry shape. The record has entries name engines plus a delivery, where the entry says entries name the files they render into, and the target set is closed and keyed by engine.
   - Evidence: the entry says "An entry is one instruction passage: a slug, its text, its place in the order, and the files it renders into, such as `CLAUDE.local.md`, Codex's own file, or both". It also requires heddle to be built "so that a further target such as `PATTERN.local.md` is easy to add later".
   - D1 makes `engines` a closed set of `claude` and `codex`, and D3 maps each engine's `file` delivery to exactly one file.
   - A file that belongs to no engine, such as `PATTERN.local.md`, would need a new axis in the model and a change to every replayed field, not just a new row. That goal point is only partly met.
   - Fix: let entries name targets from a target table, where each target carries its file name, its engine and its trigger paths. Engines and delivery then become properties of the target, not of the entry.

2. `MEDIUM` (decision), D9, where the constant lives. Question for the run's parent: the entry rules that "`lymcwd` drops its own copy" and that the name lives only in `fabricengine.BoltDirName`, with `runproc.HubOwner` "set from `BoltDirName`". The import graph (`fabricengine` imports both `lymcwd` and `runproc`) makes this impossible as worded. Is it acceptable for the one literal to sit in `lymcwd`, with `fabricengine.BoltDirName` re-exporting it and `runproc.HubOwner` reading the `lymcwd` constant?
   - This also touches `bolt-dir-rename`, whose body says it changes `fabricengine.BoltDirName`'s value. Under D9 it would change `lymcwd`'s constant instead.
   - The record already lists this under Open risks. It is filed here so it reaches the parent as a `decision` finding.

3. `LOW` (scope), D5, excluding nothing in weft. The entry says fabric seeds every listed path "when it creates a warp and weft, through the same seeding". D5 seeds warp only and says "Weft gets no line".
   - The reasoning holds, since nothing on the list is written in weft, but it departs from the entry's wording without saying so as a departure.
   - Fix: either seed both as the entry says, or record the narrowing as a deliberate reading in Auto-mode assumptions.

**Goal points of the entry**

| Goal point | Met? |
|---|---|
| No `CLAUDE.md` put into a repository | Met (Goal; it was already deleted in `c15d6390`) |
| PATTERN.md stays tracked and not in heddle | Met (Scope Out) |
| A further target like `PATTERN.local.md` is easy to add | Partly (finding 1) |
| `heddle.json` sits in Bolt beside `board.json` and is the only source | Met (D1) |
| Data kept apart from how it is shown | Met (D1, D3) |
| An entry has slug, text, order and the files it renders into | Partly (finding 1) |
| Rendering joins entries in order | Met (D3) |
| Delivery as a file or in the system prompt; Codex `developer_instructions` later | Met (D3, D4) |
| lym's own sentences stay in code | Met (D1, D4) |
| Codex target is `AGENTS.override.md`, with the repository's `AGENTS.md` first | Met (D3) |
| Re-render when a sync takes in remote commits, through a general fabric seam installed in the pre-run, reaching detached syncs | Met (D6) |
| Writes only through `lym heddle`, under the `internal/lock` write lock | Met (D2) |
| Files seeded into warp, excluded through `.git/info/exclude`, never `.gitignore` | Met (D5) |
| Excludes as their own fabric config list, separate from junctions, with fabric naming no heddle file | Met (D5); the weft half is finding 3 |
| Entries hold only what is specific to using LoomYard on the repository | Met (Constraints) |
| A re-render only when meaningful; `heddle.json` changes not pushed to live worktrees | Met (D6) |
| No watching for hand edits of `AGENTS.md` | Met (D6) |
| Two-machine writes settled by the reused entry-level replay | Met (D7) |
| Seam renamed and reshaped, with no board name or path left in fabric | Met (D7) |
| Board and heddle each register a replayer | Met (D7) |
| The by-entry merge drawn out and shared, not copied | Met (D8) |
| A re-render takes `heddle.json` as it stands; pinning a version is out | Met (D6, Scope) |
| The `lym heddle` verbs are the task's to shape | Met (D2) |
| One Bolt directory constant, keeping `_board` | Met (D9) |
| `BoltDirName` and `BoardDirName` as two names, with Bolt things placed from `BoltDirName` | Met (D9) |
| `runproc.HubOwner` set from `BoltDirName` | Partly (finding 2) |
| `lymcwd` drops its copy; messages and tests take the name from the constant | Partly (finding 2); the messages and tests half is met |
| Comments say "the Bolt checkout"; stencils and skills hold no literal | Met (D9) |

**Entries, peers and parent commits checked**

- `bolt-dir-rename`: no design conflict beyond finding 2's change of where the constant lives.
- `codex-engine`: none. D4's seam leaves room for Codex `developer_instructions`.
- `run-pins-its-version`: none. Pinning is out of scope, as the operator placed it.
- `dead-driver-false-notice`: none.
- Peer `seed-starts-at-start`: none, no shared paths.
- Peer `finalize-fix-in-place`: none, no shared paths.
- Parent commits: `lym loom peers` lists none.

**Decision notes:** no `Decision for warp-visibility:` note exists on the board list, and there are no `Added after Discussion` entries. Nothing was out of reach.

**Tool calls made:** none.

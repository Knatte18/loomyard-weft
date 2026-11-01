```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on a concrete input.

**Per-function list**

| Function or branch | Case run | Result |
|---|---|---|
| `decodeShipped` (`shipped.go:309`) | embedded `shipped.json` with one valid entry | Correct. Panics only on a defective build. |
| `Layers.Ordered` (`shipped.go:343`) | empty `Overrides`; an override for the shipped slug; a hand-edited override with empty text | The first two are correct. The third swaps in `""`, so a blank section renders (finding F1). |
| `ReadLayers` (`shipped.go:361`) | setting off; setting on with a missing override file; setting on with malformed override JSON | The first two are correct. The third returns an error, and `Render`, `Show`, `list` and `get` all fail (F1). |
| `fileText` (`render.go:225`) | shipped only; repository only; both; neither; `codex-file` with no shipped entry | Correct. The heading is skipped when no shipped text precedes it, and imports and marker order are kept. |
| `Set` and `Remove` via `refuseShipped` | slug held by the shipped layer and also present in the repository store | Wrong (F2). |
| `SetOverride` (`override.go:243`) | unshipped slug; whitespace text; replace an existing override; add a new one | Correct. A refusal leaves `s.Entries` untouched, and the clone-then-assign pattern is safe. |
| `RemoveOverride` | absent slug; stale slug no longer shipped | Correct. A stale override stays removable. |
| `overrideLayer`, `setOverride`, `requireOverrideOn` | `--layer nope`; `--layer lym` with `--order`; no `--text-file`; setting off | Correct. Each refusal comes before `readText` and before any save or sync. |
| `set` and `remove` repository path | `finishWrite` with `repositoryFile` | Correct. Error shadowing is harmless, and the save seam is kept. |
| `get` (`verbs.go:127`) | slug in both layers; unknown slug | Returns the shipped entry first (see F2). An unknown slug is refused correctly. |
| `write` and `saveLocked` | override file load failure; save failure | Correct. The lock is released by `defer`, and nothing is synced on error. |
| `LoadConfig` and `ConfigSpec` | no heddle.yaml; `lym_override: true` | Correct. The template default is false and an absent file degrades to it. |
| `claudeengine` `appendPromptText` and `init` | notice empty and non-empty | Correct. The sentence is removed once, and the creel line-break check is kept. |

**Findings**

- **F1 — MEDIUM, design.** `internal/heddleengine/shipped.go:361-376` and `:343-357`.
  - While `lym_override` is on, a malformed `heddle-lym-override.json` makes `ReadLayers` fail. This breaks every render, show, list and get for the whole hub, including the repository's own entries. The shipped layer, which the setting only tunes, should fall back to its shipped text instead.
  - A hand-edited override with empty or whitespace text bypasses the `SetOverride` check and renders a blank section under the shipped heading (`shipped.go:347-349`). `Ordered` should ignore an override whose text is blank.

- **F2 — MEDIUM, design.** `internal/heddleengine/store.go` (`Set` and `Remove` call `refuseShipped`), `shipped.go:380-385`, `replayer.go:149-161`.
  - Setting `heddle.json` entry slug S as shipped locks it out of the repository. This happens when the repository store already holds S before a lym release ships it, or when another machine's commit is replayed in, since `addEntry` calls `ValidateEntry` only.
  - `Set` and `Remove` then refuse the slug even though the repository store holds it. The entry still renders and cannot be removed or edited through the verbs. The refusal text ("give the repository's entry another slug") cannot be followed for an existing entry.
  - `get` and `list` show both entries with the same slug.
  - Fix: let `Remove` (and a `Set` on an existing entry) act on a slug the repository store already holds, or have `Remove` name a hand-edit of `heddle.json` as the way forward.

- **F3 — LOW, design.** `internal/heddleengine/replayer.go:24` and `internal/fabricengine/replay.go:675`.
  - `Replayer.Claims` covers only `heddle.json`. `heddle-lym-override.json` is a new Bolt file that no replayer claims, so a concurrent edit from two machines gets no by-entry merge. A commit touching both files also falls out of the all-claimed ledger path.
  - I did not read the Bolt sync's staging, so I could not confirm the override file is committed at all. The handler or the affected-set lens should check this.

- **F4 — LOW, design.** `internal/heddleengine/shipped.json` and `internal/shuttleengine/claudeengine/settings.go`.
  - The batching entry now reaches a session only through a rendered `CLAUDE.local.md`. A session in a worktree that is not rendered, such as one outside a hub or one whose existing `CLAUDE.local.md` lacks the marker, loses the sentence it used to get through the system prompt. This follows the decision record's channel choice. I record it as a consequence, not a defect, unless the handler judges otherwise.

**Tool calls:** none. The context already held every function I ran.

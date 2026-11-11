```yaml
lens: board-scope
outcome: reported
```

Board-scope lens review: does the decision record fit the board entry, its touching entries, its peers and any parent commits?

**Findings**

1. **MEDIUM** (scope): The record leaves out #114's first expected point, that added entries are folded and removed once folded. A Sketch's stored record can still carry appended entries.
   - Evidence: the entry's "From issue #114" Expected section says: "A decision added after Discussion is folded into the record's main decisions by the fixer, and its appended entry is removed once folded, so the record holds the current design only."
   - The record puts fixer folding under Scope "Out" and leaves folding to a manual parent `replace` (D6).
   - A motif run ends at Discussion approval, and D1 refuses `replace` before approval. So a Sketch whose Discussion took `add` calls during review is stored with its appended entries. Nothing folds them until someone runs `lym board upsert --record-file` by hand.
   - The entry's goal is "the record holds the current design only". For Sketches, the record meets it only if someone remembers to fold by hand.
   - Fix: either have D6 or the Sketch flow require a fold before or at storing, for example by telling the parent to fold with `--record-file` right after a motif run, or state the departure from the #114 point in Decisions with its rationale. Putting it only in Out is not enough.

2. **LOW** (scope): D6 tells the parent to fold with `replace`, which is refused before approval. It never names `upsert --record-file` as the fold path for a finished motif run's Sketch, which is the main case in #114 and #116.
   - Evidence: D6's text, read against D1's pre-approval refusal and D4.

**Goal points of the entry**

| Goal point | Met? | Where |
|---|---|---|
| A verb that replaces a Sketch's stored record from a file, run by the operator or parent orch | Met | D4 |
| Refused while a run holds the entry (#114) | Met | D4 |
| Raddle reads the newer record | Met | D4 together with Record-Write seeding, and D1 with D3 for a raddle run that is already held |
| Raddle validating against the body, offered as an "or" alternative | Rejected with a reason | Scope Out |
| An edit or replace verb for a recorded decision in place, committed and announced like `add`, old text kept in git history (#117) | Met | D1, as a whole-record verb; the entry's per-id `edit`/`remove` form was only "for example" |
| The same for a Sketch's stored record after its motif run (#117) | Met | D4 |
| Added entries folded and removed (#114, first point) | Partly met | Finding 1 |

**Entries, peers and commits checked**

| Item | Conflict |
|---|---|
| F14 `discussed-task-split` | None. Its rule that lym writes the record exempt from the lock holds, and D4's manual writer is refused on held entries. |
| F5 `parent-reviews-decision-record` | None |
| T7 `raddle-dep-and-loop-exit` | None. It is a different `raddle-propose` check. |
| F6 `discussion-extra-reviewer-fans` | None, judged by title only |
| Peer `machine-name-verb` | None. It touches `internal/boardengine/ownerset.go` and `doc.go`, which is a text-only package overlap. |
| Peer `worktree-create-build-leftovers` | None |
| Peer `load-turn-retry-cap` | None |
| Peer `testmap-cross-package` | None |
| Parent commits | `lym loom peers` lists none |

**Decision notes:** no `Decision for decision-record-replace:` note exists on the board, so I read none and there were none I could not reach.

**Tool calls:** none. My context already held `lym board list`, `lym board get` for this task and for F14, F5 and T7, and `lym loom peers`.

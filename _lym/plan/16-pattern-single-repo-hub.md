# Card 16 — pattern-single-repo-hub

**Create:**
- `pattern/PATTERN-single-repo-hub.md`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-run-fsm.md`
- `pattern/PATTERN-cwd-resolution.md`

**Extends:**
- `pattern/PATTERN-fabric-boundary.md`
- `pattern/PATTERN-lym-files.md`
- `pattern/PATTERN-pair-remote.md`

**Intent:** The record's PATTERN changes and the background paragraphs its Constraints list.

`PATTERN.md` gains this line under `## Fabric and git`, after `PATTERN-gitrepo-client-boundary`:

```markdown
- `PATTERN-single-repo-hub` — Running a hub as one repository: records on each code branch, the board on `lym-bolt`; merges, pull request views and code identity strip the records paths. — [background](pattern/PATTERN-single-repo-hub.md)
```

and these two lines replace the current `PATTERN-run-fsm` and `PATTERN-cwd-resolution` lines in place:

```markdown
- `PATTERN-run-fsm` — Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts, bar a prime push failing `protected`; an `operator_unpushed` push is owed; another machine's takeover hands over, never replays. — [background](pattern/PATTERN-run-fsm.md)
- `PATTERN-cwd-resolution` — Resolving a cwd, worktree root or anchor: only `lymcwd`, from the `.lym-anchor` at the worktree's records root; only `fabricengine` writes or git-reads it; engines get absolute paths, never import lymcwd; hub containers and workspace files stay hub-reachable only. (test) — [background](pattern/PATTERN-cwd-resolution.md)
```

`pattern/PATTERN-single-repo-hub.md` states the rule and its parts: the layout read from `.lym-anchor` at the worktree root (`lymcwd.OneRepository`);
records committed path-limited on the worktree's own branch (cards 7 and 10); the records paths of `RecordsPathsAt` restored to the receiving side at every fabric merge, which commits with `MergeTrailerKey` (card 9);
the records-only replay and the guarded prime push (cards 13 and 14); the Bolt on `lym-bolt`; and the tests that hold each, by name.
It names that pull request views and code identity strip the records paths too, which a later part builds.

Backgrounds:
- `PATTERN-run-fsm.md`: the rule line takes the new text, and `### An owed push holds the run` adds that an `operator_unpushed` push is owed and that a prime push failing as `protected` completes the transition with its records unpushed, held by `TestNew_ProtectedPushCompletes` and `TestPrimePushGuard`.
- `PATTERN-cwd-resolution.md`: the anchor bullet says a worktree's anchor comes from its own `.lym-anchor` at its records root, the weft root in a two-repository hub and the worktree root in a one-repository hub; `ResolveWithin` accepts a one-repository cwd at or below the anchor only.
- `PATTERN-fabric-boundary.md` gains a one-repository paragraph: the exclude gets `.lym`, the `.scratch` link and heddle's files and never `_lym`, `.lym-anchor` or the weft artifact lines; the Bolt's render-manifest `.gitignore` stays on `lym-bolt`, which no code branch merges; teardown tags and pushes the branch tip, deletes the branch and the `lym-view/` ref.
- `PATTERN-lym-files.md` gains one: `_lym` and `.lym` are real directories at the anchor, and records are per-branch and never a merge participant because fabric's merge restores them to the receiving side.
- `PATTERN-pair-remote.md` gains one: a one-repository pair pushes its one branch once; the prime's records push is guarded and owed as `operator_unpushed` while the operator has unpushed commits.
The overview lines of `PATTERN-fabric-boundary`, `PATTERN-lym-files` and `PATTERN-pair-remote` stay.

**ImpactSummary:** PATTERN.md gains one entry and rewrites two; three background files gain a paragraph and two rewrite one; no code changes.

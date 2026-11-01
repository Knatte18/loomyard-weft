```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attacking the record's claims and bounds against concrete counterexamples.

**Claims attacked**
- D1, unknown keys kept by the store's own types, as `storeFile` and `Task` do: held.
- D1, "an absent file holds no entry", when the file is missing only briefly during a Bolt checkout: **broken** (F1).
- D2, a heddle write serializes with Bolt's pull and replay through the board write lock: held for writers. Renders take no Bolt lock (F1).
- D2, the board's sync commits whatever is dirty in Bolt: held (`boardengine/sync.go` `commitDirty`, `Bolt.Commit`).
- D3, an in-place write is safe because "a session reads only when it starts": **broken** by compaction (F2).
- D3, the per-worktree lock makes the last render's text stand whole: held between renders. It does not cover readers (F2).
- D3, the `@AGENTS.md` import keeps the repository's `AGENTS.md` reaching Claude: **broken** for a repository whose only `AGENTS.md` is `.claude/AGENTS.md` (F4).
- D3, Claude Code's AGENTS.md rules (a `CLAUDE.local.md` counts, the import is expanded, a file is not read twice): held. The docs table and the `claude-md-or-agents-md` row match. No `--setting-sources` on shuttle's lines, so `CLAUDE.local.md` loads.
- D3, the marker check protects a file the repository tracks: held in practice. A tracked file would have to carry heddle's marker to break it, which is contrived.
- D4, a resume through the launch's own line keeps its passages, and an adopted resume reads heddle afresh: held (`Prepare` with `ResumeSessionID`; the resume line carries `--append-system-prompt-file`).
- D4, outside a hub or with no seam installed, no passage: held (`c.dirs.Board` empty; the pre-run installs only when `!testing.Testing()`).
- D5, `info/exclude` is one file for every worktree of the warp repository: held (`resolveGitExcludePath` uses the common dir).
- D5 and D3, lines seeded before any render writes: held as specified.
- D6, a detached process reaches the seam, "such as the board's Bolt sync": **broken**. The Bolt sync never moves a code branch (F3).
- D6, a records or Bolt fast-forward fires nothing: held, and it is what breaks the example above.
- D7, the tag `set-aside/replayer/&lt;name&gt;/&lt;sha&gt;` cannot be mistaken for a run's: held. Bolt shares the records repository with the wefts, but `SnapshotTagged` counts only annotated tags, and replay tags are lightweight (`snapshotuncommitted.go:106-121`).
- D7, the board's sync commits everything dirty in Bolt in one commit, so a mixed commit is possible: held.
- D7, the ended-set and owner-read rewiring: held. The weft ended set (`pull.go` `weftEndedByOrigin`) also reads `InstalledBoardReplay().Entries`; that is covered by "ended set".
- D9, the new `runproc` → `lymcwd` edge closes no cycle: held (`lymcwd` imports only stdlib, `dotgit`, `weftname`).
- D9, `PATTERN-leaf-packages`'s background "admits it": **broken as stated today** (F5).
- D9, `_board` left only in one declaration and the policed-token list: holds only if the owner-row key and planted fixtures are also rebuilt (F6).

**Findings**

F1 — MEDIUM — design — D3/D6 render inputs; D1 "An absent file holds no entry"
A render reads `heddle.json` without the Bolt write lock. Any git move of the Bolt checkout over a `heddle.json` change replaces the file by unlink-then-create, so a concurrent render can find it absent. Moves that do this: Bolt `Pull`'s fast-forward (`bolt.go:307` `MergeFFOnly`), a replay's cherry-pick, and `ffWorktree` inside `reconcileRecordsBranch` (`originreconcilemoves.go:197`).
Counterexample: `Topology.Add` fires "made" in one process while `lym board` pull-first in another fast-forwards Bolt over a heddle commit. Under D1 the render sees no entries. Under D3 it writes no file, and on a take-in render it removes the existing marked `CLAUDE.local.md`. No warning is raised, since an absent file is a valid state.
Board writes are atomic (`boardengine/render.go:46`, `replay.go:468` use `fsx.AtomicWrite`), so heddle's own saves are not the risk; git's checkout is.
The record bounds renders against each other (D3 render lock) but never against Bolt's writers.
Evidence: `fabricengine/bolt.go:307`, `fabricengine/originreconcilemoves.go:155,197`, D1, D3.

F2 — MEDIUM — design — D3 "It writes the file in place, which a session reads only when it starts"; support log Turn 4 bound-breaker 5; rejected alternative "An atomic write by a temporary file renamed into place"
False premise. The Claude Code memory docs say: "after `/compact`, Claude re-reads it from disk and re-injects it into the session". `CLAUDE.local.md` "loads alongside `CLAUDE.md` and is treated the same way".
Counterexample: the prime orch session lives in the prime worktree. A `lym fabric pull` there takes in an `AGENTS.md` change, D6 re-renders `CLAUDE.local.md` in place, and the orch compacts at the same moment. It can re-inject a truncated or half-written file. Even without the race, a live session's instructions change at its next compaction, so "read only at start" is wrong.
The in-place choice and the rejection of rename-into-place both rest on this premise. A temporary file at an excluded path, or under `.lym` on the same filesystem, would not show as untracked.
Evidence: code.claude.com/docs/en/memory, "Instructions seem lost after `/compact`" (fetched docs line 653) and line 123.

F3 — LOW — consistency — D6 "In a detached process, such as the board's Bolt sync, the failure is log-only"; D6 acceptance "a failing seam in a detached Bolt sync logs a warning naming the worktree and `lym heddle render` and leaves the move intact"
The board's Bolt sync moves only the Bolt checkout: `Sync` runs `commitDirty` then `Bolt.Push`, and `PushRecorded` retries only through `retryBoltPush`. D6 also says a Bolt move fires nothing.
So the cited detached process never fires the take-in event, and the acceptance case cannot be built as written. A detached process that does move warp has to be named instead, if one exists. Foreground warp movers seen: `fabriccli` pull, `PullHub`, `battencli`, the `ReconcileOrigin` callers.
Evidence: `fabricengine/bolt.go:104-128`, `boardengine/sync.go:35-58`, D6 second bullet.

F4 — LOW — design — D3 `claude-file` row: "an `@AGENTS.md` import line when the worktree's root holds an `AGENTS.md`"
Claude Code also reads `.claude/AGENTS.md`: "every `AGENTS.md` and `.claude/AGENTS.md` in your working directory and the directories above it" (docs line 375). A `CLAUDE.local.md` counts against it the same way it does against a root `AGENTS.md` (line 370).
Counterexample: a repository that keeps only `.claude/AGENTS.md` and no `CLAUDE.md`. The render writes `CLAUDE.local.md` with no import, and Claude stops reading the team's instructions. That defeats D3's stated purpose. The re-render paths omit `.claude/AGENTS.md` as well.
Evidence: memory docs lines 370 and 375; D3 row and rationale.

F5 — LOW — consistency — D9 bullet 2: "PATTERN-leaf-packages's background admits it, as Constraints states, and its overview line under `## PATTERN changes` stays as it is"
The background today admits nothing new: "`internal/runproc`: Imports only the standard library, `internal/proc`, `internal/state` and `internal/logger`." The claim holds only once the task edits the background. As written it reads as a fact about the current file, and nothing in the record says the background file itself must change.
Evidence: `pattern/PATTERN-leaf-packages.md` `internal/runproc` section; D9; Constraints.

F6 — NIT — consistency — D9 acceptance: "the `_board` literal appears in one Go declaration and in `TestEnforcement_GeometryLiterals`'s policed-token list"
The same test also spells `"_board"` as an owner-row map key and inside its planted fixture sources (`filepath.Join_arg_board`). The criterion fails unless those are rebuilt from the constant too, which D9's exception for the token list alone does not say.
Evidence: `internal/lymcwd/enforcement_test.go:209,225,412-413`.

**Tool calls**
1. `snapshotuncommitted.go:104-130`, `bolt.go:100-135` and the warp-moving callers: to test whether replay tags clash with `SnapshotTagged`, and whether the Bolt sync ever moves warp. The context lacked `SnapshotTagged`'s filter and `PushRecorded`.
2. `hubpull.go:120-145` and `PullHub` callers: to find which processes move warp one hop past `Pull`. The context lacked them.
3. `boardengine` save functions (empty result): to find how `board.json` is written. The context lacked the store's writer.
4. `boardengine` write calls: to confirm board writes are atomic, which places F1's risk on git's checkout rather than on heddle's saves.

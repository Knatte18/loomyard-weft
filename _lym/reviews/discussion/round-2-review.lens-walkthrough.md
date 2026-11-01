```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, checked against the decision's own stated purpose.

**Decisions walked**

- **D1:** `lym heddle set a` on an absent `heddle.json`, then `set b`, then `remove a`, where a newer binary had added an unknown key to `b`. The types keep their own unknown keys, so `b` keeps its key. Purpose met.
- **D2:** `lym heddle set foo` with text, order and `claude-file` in a task worktree. It saves under `board.lock`, then the detached `lym board sync` runs `commitDirty` (which stages everything dirty) and `Bolt.Push`. Purpose met. No warp worktree is touched.
- **D3, `claude-file`:** a root holding `AGENTS.md` and no `CLAUDE.md`, with the anchor at `svc/`. The render writes `CLAUDE.local.md` with the marker, `@AGENTS.md` and the entries. Claude reads it from the root, and the import resolves relative to the file. Purpose met.
- **D3, `claude-file`, AGENTS.md under `.claude`:** a repository whose `AGENTS.md` sits at `.claude/AGENTS.md` and not at the root. No import is written, but the render's `CLAUDE.local.md` still counts against AGENTS.md loading, so Claude stops reading the team's file. Purpose missed (F2).
- **D3, `codex-file`:** a root `AGENTS.md` plus one `codex-file` entry. The override is AGENTS.md followed by the entry, and Codex loads the override alone. Purpose met.
- **D4:** a fresh launch in a hub, where `c.dirs.Board` is the Bolt checkout under D9. The seam returns the `claude-prompt` passages, which are appended after a blank line. An adopted resume runs `Prepare` again and reads heddle afresh. Outside a hub, `Board` is empty and no passage is added. Purpose met.
- **D5:** `Topology.Add` with the template default. `/CLAUDE.local.md` and `/AGENTS.override.md` are seeded once into the shared common-dir exclude through `mutateGitExclude`. Purpose met.
- **D6, "made":** `Topology.Add`, then wiring and exclude seeding, then "made", then a render. Purpose met.
- **D6, "took in":** `lym fabric pull` where warp moves A→B and B changes `AGENTS.md`. `ResetHard` runs, the paths are `[AGENTS.md]`, and both file targets re-render. Purpose met.
- **D6, detached take-in:** the record's own example, the board's Bolt sync. `Sync` → `commitDirty` + `Bolt.Push` → `pushClassified` with `retryBoltPush` reaches only the Bolt branch. That process never moves a warp code branch, so the case never runs (F1).
- **D7:** a sync commit touching `board.json` (an ended run's entry) and `heddle.json`, with origin moved. Both replayers apply. The ledger part is skipped, the heddle part lands in one commit, and the original is tagged. Purpose met.
- **D8:** the same slug changed on both sides in the `text` field. Origin's value is kept, the loss is reported, and the unknown key stays. Purpose met.
- **D9:** `BoltDirName` = the `lymcwd` constant. `BoardDirName`, `BoltDir`, `BoardWriteLockPath` → `BoltDir`, and `HubOwner` all derive from it, so `bolt-dir-rename` changes one value. One acceptance check fails as written (F3).

---

**F1: MEDIUM, design, D6 and acceptance D6. The detached process D6 names never fires the seam.**
D6 says "In a detached process, such as the board's Bolt sync, the failure is log-only". Acceptance D6 asks that "a failing seam in a detached Bolt sync logs a warning naming the worktree and `lym heddle render` and leaves the move intact". The Bolt sync only commits Bolt and pushes it, replaying onto origin's Bolt tip, and D6 itself says the event never fires "for a records or Bolt branch". That acceptance case has no mechanism to exercise. The detached or daemon processes that do move warp code branches go unnamed: `ReconcileOrigin` from batten's cancel, resume and wire paths and from hibernate.
Evidence: `internal/boardengine/sync.go:41-57` (`Sync` step = `commitDirty` + `bolt.Push`); `internal/fabricengine/bolt.go:104-128` (`Push` → `pushClassified(..., failedPushSideBolt, ...)` with `retryBoltPush`); `ReconcileOrigin` callers in `internal/battencli/cancelcmd.go:337`, `internal/battencli/resumereconcile.go:36` and `internal/hibernatecli/reconcile.go:46`.
Fix: name a process that actually moves a warp code branch while detached, or state the rule without the Bolt sync example, and point the acceptance case at that process.

**F2: LOW, design, D3 (`claude-file` row and import condition). The `@AGENTS.md` import condition ignores `.claude/AGENTS.md`.**
Claude reads "every `AGENTS.md` and `.claude/AGENTS.md` in your working directory and the directories above it". `CLAUDE.local.md` counts against that read. A render in a repository whose only AGENTS.md is `.claude/AGENTS.md` therefore hides the team's instructions from Claude, which D3's rationale says the render must prevent. The re-render paths leave `.claude/AGENTS.md` out as well.
Evidence: Claude Code memory docs (fetched page, the "When Claude Code reads AGENTS.md" section: lines 370 and 375 of the saved tool result); `decision-record.md` D3 `claude-file` bullet.
Fix: import whichever of `AGENTS.md` or `.claude/AGENTS.md` the root holds, and add the path to the re-render list. Or record the gap under Open risks.

**F3: LOW, consistency, D9 acceptance. The `_board` literal is also needed outside the policed-token list.**
The acceptance criterion says the literal "appears in one Go declaration and in `TestEnforcement_GeometryLiterals`'s policed-token list". The same test also holds the literal in its owner-row key and in planted fixture sources, such as `filepath.Join(x, "_board")`, which exist to show the scan catches it. Taking those from the constant would also empty the `_boardroom` near-token case of meaning after the rename.
Evidence: `internal/lymcwd/enforcement_test.go:209`, `:225`, `:412-413` and `:462-464`.
Fix: widen the exception to the whole test, its token list, owner row and planted fixtures included.

---

Tool calls:
1. `internal/fabricengine/bolt.go` lines 100-135, plus a grep of `ReconcileOrigin` callers and of warp-move calls in the Bolt push files. My context did not show what `Bolt.Push` reaches, or which processes move warp code branches, and I needed both to check D6's detached-sync case.

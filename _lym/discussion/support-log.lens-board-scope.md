```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: checked the record against the board entry, its peers and the parent commits.

**Findings**

1. **MEDIUM, class `decision`. Applies to the Goal and to D1–D3.** A parent commit, `7c23591` ("Show a beam's pawl checklist in a second overview window", 2026-11-03), fixes this same bug with a different design.
   - What it did:
     - reed allows several overview strands, ordered by a new `Display.Order`;
     - `statusStrandAddSpec` always uses the overview anchor, titled with the recipe;
     - `ensureRunsStrand` no longer replaces the status strand.
   - It left out the overview line that D3 now adds.
   - It is not an ancestor of this worktree's HEAD (`git merge-base --is-ancestor` exits 1), but `lym loom peers` lists it among the parent commits. The likely reading: this task is a bench arm replaying the reference run from its pre-landing base, and that commit is the reference's own landing.
   - Question for the run's parent: should the record treat `7c23591` as out of view? The bench's frozen base predates it, so the D1–D3 design stands as an independent arm. Or should it align with the landed design or cite it?
   - The record states that reed refuses a second overview strand, keeps that refusal as a Constraint, and puts reed changes out of scope. All three hold at this base and contradict the landed commit.

2. **LOW, consistency. Applies to Scope Out.** "Reordering `lym beam start` or the pawl recipe" is put out of scope. The entry offers that reorder only as an alternative ("or the beam start brings the runs strand up before…"), so leaving it out is a valid pick, not a dropped goal. No fix needed. Noted only because the support log already gives the reason.

**Goal points of the entry**

- Checklist in a window of its own beside the beam overview while pawl lives: met by D1 and D2.
- Window named for pawl: met by D2, which takes the recipe from the seed, so a pawl run's window is `pawl`.
- When the runs strand takes the overview window, the checklist is added back in its own window: met by D1.
- One line in the beam overview with pawl's current row, pointing at the window: met by D3.

**Entries, peers and commits checked**

- `wxb-plan-with`, the sibling bench arm on the same body: no conflict, it is a parallel arm by design.
- `plan-writer-explorer-forks`, the parent bench: no conflict. Its changed paths are only under `docs/research`.
- `fabric-single-repo-mode`: no conflict.
- `testmap-uses-design`: no conflict.
- Board notes such as B1 `board-view-not-refreshing`: no touch.
- Parent commit `7c23591`: conflict, see finding 1.
- Parent commit `1884128` ("Beam reed session gets the beam button group") was only seen in the list, not read: no apparent conflict.

**Decision notes:** `lym board list` shows no `Decision for wxb-plan-without:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

- `lym loom peers`: my context lacked the peer list and the parent commits.
- `git show --stat 7c23591` plus `merge-base --is-ancestor`: the peers list named a commit with this task's subject, and my context lacked its content and whether HEAD contains it.

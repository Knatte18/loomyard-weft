```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of decisions that act on the same state, and each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (replace's trailer commit gives D2 its cut point): **broken**. The named seam makes no commit when nothing is staged, so the trailer can be missing (F1).
- D1 failure path × D2 (restore on a failed commit after a sweep already landed the replaced bytes): **broken** (F2).
- D1 × `add` under the per-run decision lock (base hash, ordering): held.
- D1 × D3 (replace refreshes a stored record on a raddle run through `StoreDecisionRecord`, which bypasses the run lock): held, provided the commit lands.
- D3 × `add` on a Record-Write run after a replace (`checkDarnRecord` runs `CheckAddedDecisionsKept` over `history`): **broken** when the trailer is missing (F1).
- D2 × a beam's project record and its member pairs (the trailer names no record): **broken** (F3).
- D2 × an `add` after a replace that a writer then drops (AC D2 third bullet): held, because versions after the cut include the entry.
- D4 × D2 (a board-side fold leaves no trailer, then a raddle run on the same slug reads the run's history): **at risk** (F4).
- D4 × D1/D3 (way forward from a held entry or live batten run): held. The chain ends at `replace`, which before Record-Write refuses with `start`/`resume`, and the seed then brings the board record in.
- D4 × D5 (record and body in one write, stamp = body that write leaves): held.
- D5 × D3 (`StoreDecisionRecord` stamps on a run copy): held. It is a no-write when the bytes are unchanged, so the stamp only refreshes on a real store, which is harmless.
- D5 × Raddle-File (`convertRunEntry` rewrites the body through `arrasProjectOf`; "body as that write leaves it"): held.
- D5 × `MakeSketch` and `promoteTo` (neither changes the body): held. AC D5 second bullet is consistent.
- D6 × D1/D2/D4 (spec and directive name replace, the history reset and `--record-file`): held.
- Open risk 3 × D3: held. The risk is not settled by a Decision, and `seedDiscussedRecord` does overwrite (`internal/loomcli/discussionrecord.go:71`).
- Constraints `PATTERN-fabric-boundary`, `PATTERN-lym-files` and `PATTERN-sole-parsers` against D1, D2 and D4: held.
- AC D2 second bullet against D1: **unreachable** through the named seam (F1).

**Findings**

**F1 — BLOCKING, design, D1 (commit) × D2 (cut) × AC D2 second bullet**
D1 says the trailer commit "is made even when the record's bytes already landed in an earlier commit", through `fabricengine.CommitAnchoredPathsWithTrailers`. That seam calls `CommitAnchoredPaths` and then `gitrepo.Repo.StageAndCommit`. When `git diff --cached --quiet` is clean, `StageAndCommit` returns `committed=false` and makes no commit.
- So when a sweep already landed the bytes, no trailer commit exists.
- D2 then finds no trailer, or an older one, and reads the history whole, so the dropped entries come back as `added-decision-dropped`.
- On a Record-Write run, every later `add` is refused by `checkDarnRecord` with "missing an entry its history held", which undoes the replace.
- The only empty-commit primitive is `gitrepo.Repo.CommitEmpty`, wrapped only by `commitEmptySnapshot`, and it carries no caller trailers.
- The record claims an existing seam does what it does not, and does not name the new one it needs.
- Evidence: `internal/gitrepo/gitrepo.go:146-151`, `internal/fabricengine/trailercommit.go:77-103`, `internal/fabricengine/recordsgit.go:180`, `internal/loomcli/decision.go:470-476`.

**F2 — MEDIUM, design, D1 failure path × D2**
"When the commit fails, it restores the prior bytes and refuses." If a sweep commit landed the replaced bytes before the failure, HEAD holds them with no trailer while the working copy goes back to the prior bytes.
- The working copy then differs from HEAD, so the next `add` or `replace` is refused (`decisionWayDiffers`).
- HEAD's version lacking the added entries is unbounded by any trailer, so the gate flags them.
- `add` handles the same case by reading HEAD after a failed commit (`afterFailedCommit`, `committedLacksEntry`). D1 states no such check, and the way forward ("re-run replace") fails its own base check, because HEAD no longer hashes to the parent's base.
- Evidence: `internal/loomcli/decision.go:650-680`, `internal/loomcli/decision.go:882-891`.

**F3 — MEDIUM, design, D2 trailer keying**
The trailer carries only `--by` and a sha256, not the record's path. `FindTrailerCommit` takes the newest commit on the first-parent walk that carries the key, whatever path it touched.
- A pair forked from a beam's weft branch carries the beam's commits, including a project-record replace trailer.
- D2 then picks a trailer whose hash matches no version of the member's own record, and the record does not say what the history returns in that case (empty, or whole).
- An empty history would also silence the dropped-entry check for every earlier entry.
- Evidence: `internal/fabricengine/trailercommit.go:116-147`, `internal/loomcli/projectrecord.go:39-58`, `internal/fabricengine/add.go:147-153`.

**F4 — LOW, design, D4 × D2 (unverified premise)**
A fold made with `upsert --record-file` writes no trailer. The record does not say whether a raddle run of the Sketch starts on a fresh weft history or adopts the motif pair's live weft branch.
- `fabricengine.Add` adopts a weft branch that is live, meaning local, or on origin with no archive tag.
- If adopted, `seedDiscussedRecord` lands the folded record over versions that hold added entries, and D2 finds no trailer, so the gate flags every folded entry.
- Record-Write instead keeps an existing record file and never reads the fold.
- I did not verify whether the motif pair's teardown archives its weft. The record should state which history a raddle run starts from, or key the cut so a board-side fold also bounds it.
- Evidence: `internal/fabricengine/add.go:150-153`, `internal/loomcli/discussionrecord.go:71-81`, `internal/loomshed/recordwrite.go:124`.

**Tool calls**
1. grep in fabricengine and shedrecipe for weft-branch reuse and the `SeedDiscussedRecord` gate: the manifest did not cover how a new pair meets an existing weft branch.
2. grep for pairteardown archive tags and `MakeSketch` callers, and `internal/fabricengine/add.go:140-160`: needed to see whether a Sketch's motif weft history is adopted by a later run on the same slug.

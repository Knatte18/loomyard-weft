```yaml
lens: fix-proposal
outcome: reported
```

Fork report (lens fix-proposal): I challenged each finding the fixer carried out as the reviewer proposed. I found one LOW issue and nothing worse.

**LOW. F6 head record, `internal/hardenshed/round.go` `startHead`.**
- **Change:** a fix row now reuses the first head recorded in `harden-head-<round>.json` on every later entry of the same round.
- **Case where it is wrong:** `harden-head-<round>.json` is written at the first entry and never cleared. If the first call's commits are rewritten (the fixer amends, resets or rebases) and the row is then re-entered, the recorded head is no longer an ancestor of `HEAD`.
- **Result:** `Commits(head)` runs `CommitLog(from, "HEAD")` over a dangling base. Either the call errors, which `writeCommits` reports through `fileError` and `ArchiveRound` is skipped for that row, or the log lists far more commits than the round made.
- **Evidence:** the head record is read before `r.deps.Head()` on every call, and nothing in `roundRow.Call` removes or revalidates it.
- **Test gap:** `ResumedFixRowKeepsTheHeadItFirstRecorded` only covers a head that is still valid.
- **Impact:** a rare edge. It does not stop a normal crash-resume, which is the case the finding asked for.

**No finding on the other proposals.** For each, I looked for a case where the proposal is wrong.
- **F5, empty reviewer or fixer list:** the template ships `reviewer` and `fixer` as three-entry lists, so a default hub does not trip the new refusal.
- **F1, archive under the anchor:** I built no failing case. `Campaign.Commit` goes through `CommitAnchoredPaths` and the archive now sits under the anchor, so the two agree.
- **F7, vet and guard steps in a `--copies` call:** I found no breakage in what the new condition covers.
- **F13, `lymBinary` seam:** it is set in `New` and the only caller of `clearWorktreeSandbox` is the field assigned there, so no `Teardown` literal reaches it unset.
- **F11, `greenForCall`:** the extraction is a pure move.
- **F15, Harden-Burler-Fix in the Verify sites sentence:** the fixer added only the Verify-site mention and left out the `live` gate the review named. That is right, since `NewLiveGate` runs `lym gate test` and not `Verify`.

Tool calls:
1. `sed` of `internal/hardenshed/template.yaml` lines 1–14: to see whether the default `reviewer` and `fixer` lists are non-empty, which my context lacked.

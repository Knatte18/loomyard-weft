```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that write, reset or read each piece of state D1 to D5 introduce or change.

**Paths traced, per decision**

- **D1, the replace commit through `CommitAnchoredPathsWithTrailers` → `CommitRecordsPaths` → `StageAndCommit`:** breaks. Nothing staged means no commit, so no trailer.
- **D1 against a sweep commit of `_lym` that lands the replaced bytes first (status commits, `commitDiscussion`):** breaks. The replace's own commit is then a no-op without a trailer, and the restore after a failed commit leaves a refusal loop.
- **D1 against the Bouncer carry-over writer (`writeCarryOver`, `wiring.go:512-519`, reached from `shedadapters/bouncer.go:760,1198,1318`):** breaks, if a settle can write after approval. The writer is outside the new decision lock.
- **D1 against `AppendDecisions` (`add`) and `decision store`:** survives, since they share the new lock.
- **D1 against Record-Write `Call`:** survives. It never overwrites an existing record (`recordwrite.go:124,153`).
- **D1 against `seedDiscussedRecord` on re-entry:** survives when D3's copy succeeds. The failure case is the stated Open risk.
- **D1 against `IdentityText` review subjects (`subjects.go:77`):** survives. The record states the re-review.
- **D2, `recordHistory` callers (`add`'s `checkDarnRecord`, the discussion gate at `wiring.go:418,881`, Darn-Write's guard at `darnwiring.go:165`, `validate-discussion`):** survive, and read the bounded history as intended.
- **D2 against pair weft forks that inherit the parent's records history (`fabricengine/doc.go:927-952`) and `FindTrailerCommit`'s key-only walk (`trailercommit.go:120-146`):** breaks. The trailer names no record path.
- **D3 against `copyRecordToBoard` and `StoreDecisionRecord`'s run-lock bypass:** survives.
- **D3 against Raddle-File's family filing (`convertRunEntry`, `raddlefile.go:204-228`):** breaks. A later add or replace on a non-perch raddle run copies the Sketch record over the family project's record.
- **D4 against `checkRunLocks`, Record-Write and `seedDiscussedRecord`:** survives. The upsert is refused while a run holds the entry, and the readers pick up the new file.
- **D4 against board replay (`replay.go:49-61`, which claims `decision-&lt;slug&gt;.md`):** survives.
- **D5 against `MakeSketch`/`sketchOf` and `promoteTo`:** survives. Neither touches the body (`sketch.go:49-60`, `store.go:811-826`).
- **D5 against `convertRunEntry`'s body rewrite:** survives, because the same write stamps the hash.
- **D5 against the no-write shortcut in `StoreDecisionRecord` (`decision.go:54`):** survives. An unchanged record keeps its stamp.

**Findings**

1. BLOCKING · design · D1, "The commit is made even when the record's bytes already landed in an earlier commit, so the trailer always exists"
   The seam D1 names makes no commit when nothing is staged.
   - `CommitAnchoredPathsWithTrailers` delegates to `CommitAnchoredPaths` (`internal/fabricengine/trailercommit.go:77,103`).
   - That reaches `gitrepo.Repo.StageAndCommit`, which returns `committed=false` with no commit when `git diff --cached --quiet` is clean (`internal/gitrepo/gitrepo.go:146-151`).
   - The only empty-commit path is `commitEmptySnapshot` → `CommitEmpty` (`internal/fabricengine/recordsgit.go:178-187`). It carries no caller trailers and refuses a staged index (`gitrepo/doc.go:466`).
   - So when a sweep commit already landed the replaced bytes, no trailer commit exists. D2 then reads the whole history, and the dropped entries come back as `added-decision-dropped` findings in every gate. That is exactly the case the record says it handles (acceptance criterion "D2: when the replaced bytes landed in an earlier commit…").
   - D1 has to name a new fabricengine seam for an empty, trailer-carrying commit and state how it behaves with a staged index.

2. MEDIUM · design · D2, "finds the newest commit on the records branch that carries that trailer for the record"
   Nothing in the trailer ties it to one record.
   - The trailer value is only `--by` plus a sha256, and `FindTrailerCommit` matches on the key alone (`trailercommit.go:120-146`).
   - A pair's weft forks from its parent's records branch (`fabricengine/doc.go:927-952`), and only shed run records are dropped. A member pair under a beam therefore inherits the beam's commits, including a replace of the beam's project record.
   - On a member that never replaced its own record, the newest trailer found is the beam's. No version of the member's record hashes to it.
   - D2 does not say what the history then returns. If it is empty, the dropped-entry check on the member stops guarding anything; if it is whole, the bound is silently lost.
   - The trailer needs to name the record's anchor-relative path, and D2 has to state what happens when no version matches.

3. MEDIUM · design · D3, "or when the run's board entry carries a stored record"
   On a raddle run's entry after a family filing, D3 overwrites the project's record.
   - `FileRaddle`'s family filing turns the run's entry into the project and writes `ProjectEntry.Record` as `decision-&lt;run-entry&gt;.md` (`internal/boardengine/raddlefile.go:204-228`).
   - The entry then still carries a stored record. `perchFilingRefusal` refuses only perch mode (`internal/loomcli/decision.go:942-950`).
   - An `add` or `replace` on a non-perch raddle run standing at or past its RaddleFile row would therefore copy the run's Sketch record over the filed project's record.
   - Today the raddle run is never past a Discussion approval, so no copy happens. D3 introduces the overwrite and says nothing about it.

4. MEDIUM · design · D1, "When the commit fails, it restores the prior bytes and refuses; the way forward is to re-run `replace`"
   A swept commit followed by a failed replace commit leaves a refusal loop.
   - A status or discussion sweep can land the replaced bytes, without a trailer, before the replace's own commit. `commitDiscussion` stages the whole directory (`internal/loomcli/wiring.go:441-467`), and the new lock does not cover sweeps.
   - If the replace's commit then errors, restoring the prior bytes leaves the working copy different from HEAD.
   - A re-run reads that working copy and passes its hash, which is not HEAD's. It is refused by D1's own "working copy equals that committed copy" check, so the named way forward cannot succeed.
   - `add`'s `afterFailedCommit` (`decision.go:657-680`) shows the case the code already handles for amendments; D1 has to handle a HEAD that already holds the written bytes in the same way.

5. LOW · design · D1, "`add`, `replace` and `store` take one per-run decision lock … so a base check and the write it guards are atomic"
   The Bouncer's carry-over writer is outside the lock.
   - It writes the record and commits it (`internal/loomcli/wiring.go:512-519`, reached from `internal/shedadapters/bouncer.go:760,1198,1318`).
   - If any settle writes a carry-over block while the run is past Discussion approval, it can land between the replace's base check and its write. The replace then drops that block, which D1 says must be kept byte for byte.
   - I did not confirm whether a settle after approval reaches this seam. D1 should either list the writers its lock excludes or bring the carry-over write under the lock.

**Tool calls**

1. grep for `writeCarryOver` callers and amendment-record readers: the context lacked the carry-over writers' call sites.
2. ls and grep of the raddle recipe in `internal/loomrecipe`: needed for the rows after RaddleFile. It found only a test file, so this stayed unconfirmed.
3. grep of `fabricengine` and `beamcli` docs for weft fork inheritance: the context lacked how a pair's records branch is forked.
4. `sed` of `fabricengine/doc.go:920-960` and grep of `loomshed/generation.go`: to confirm the fork carries the parent's records apart from run records, and to find the amendment readers.
5. grep of `settledguard.go` and `wiring.go` for amendment-title matching: inconclusive, so no finding was raised from it.

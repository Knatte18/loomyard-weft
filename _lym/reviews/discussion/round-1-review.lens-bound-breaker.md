```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and code claim the record states, looking for an input or interleaving that breaks it.

**Claims attacked**
- D1, commit "made even when the record's bytes already landed, so the trailer always exists": **broken**.
- D1, `CommitAnchoredPathsWithTrailers` "already takes caller trailers": held.
- D1, the per-run decision lock makes the base check and write "atomic against a concurrent add or replace": **broken**, because other record writers never take the lock.
- D1, "when the commit fails, it restores the prior bytes and refuses; way forward re-run replace": **broken** when the commit landed but the call errored.
- D1, "A replace changes `IdentityText` … reviews it again": **broken**.
- D1, kept carry-over blocks bound the replace: **partly broken**. Blocks are only checked as kept; adding one is unchecked.
- D1, the notice tells "each live strand … to re-read it whole": **broken** for Merriam.
- D1, the caller guard ("only the parent, its arbiter or the operator's unnamed shell"): held, but the list omits the sampler strand that `add`'s sampler rule admits with `--by operator` (`decision.go:736-740`).
- D2, "Only a lym verb writes a commit message on the records branch" (the trailer can't be forged): held as far as I checked. `lym fabric merge -m` only writes merge commits.
- D2, the cut at the newest version whose hash matches the trailer: held, given a trailer exists. It falls with the first D1 claim.
- D2, "Every caller of the history seam reads the bounded history": held. All callers go through `recordHistory` (`decision.go:1121`, `wiring.go:418,881`, `darnwiring.go:165`).
- D3, `StoreDecisionRecord` "already bypasses the run lock for the record and flag alone": held (`boardengine/decision.go:42`).
- D4, "the board's existing classification of a run's agent": held (`taskguard.go:14` `isRunAgentSession`).
- D4, refused for "an entry whose slug has a live batten run": **broken**. It is not atomic, can't see another machine, and names no seam.
- D4, the record and payload "land in one board write … as `StoreDecisionRecord` does": **broken** on a payload the store refuses after the file write.
- D5, "the Sketch conversion, label edits and promote's own write leave the body alone": held (`sketch.go:49-59`, `store.go:811-826`).
- Constraint `PATTERN-lym-files`, the lock path via `internal/lock`: held (`lock.TrackedFileLockPath` exists).

**Findings**

1. **BLOCKING · design · D1 commit path and D2 cut.** No trailer commit is made when the replaced bytes are already committed.
   - Counterexample: `replace` writes the file. Before its own commit, a sweep commits the discussion directory without the trailer. Sweeps include a loop row's `CommitDiscussion` or a Plan/Code-Review Bouncer's `writeCarryOver`, which commits the record alone.
   - `replace` then calls `CommitAnchoredPathsWithTrailers`, which hands off to `CommitAnchoredPaths` → `StageAndCommit`. That sees nothing staged and returns `committed=false` with no commit.
   - With no trailer, D2 reads the whole history, and the next gate reports `added-decision-dropped` for every entry the replace folded.
   - No fabric seam makes an empty commit with trailers. `CommitEmpty` is reached only through `commitEmptySnapshot`, and it refuses a non-empty index.
   - Evidence: `internal/gitrepo/gitrepo.go:146-151`, `internal/fabricengine/trailercommit.go:77,103`, `internal/fabricengine/recordsgit.go:180`, `internal/loomcli/wiring.go:514-519`.

2. **BLOCKING · design · D1 decision lock.** The lock covers only `add`, `replace` and `store`. Writers that run beside a past-approval `replace` never take it.
   - These writers are the Plan-Review and Code-Review Bouncer settles. They write carry-over blocks into the decision record through `writeCarryOver` and commit it, and the tapestry and sampler recipes set `carry_over` on those rows.
   - Interleaving A: the settle reads the record, `replace` checks its base and writes, then the settle writes the old record plus its block. The replace is silently lost, after the verb reported success.
   - Interleaving B: the settle writes a block after `replace` checked the carry-over set but before it wrote. The replace then overwrites a Go-only block, which D1's "kept carry-over blocks" bound claims cannot happen.
   - Evidence: `internal/shedadapters/bouncer.go:760,1198,1318`, `internal/loomcli/wiring.go:1010`, `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:321,501`, `contracts/recipes/sampler-recipe.yaml:140`.

3. **MEDIUM · design · D1 failed-commit path.** `StageAndCommit` can return an error after `git commit` landed, when `CurrentSHA` fails.
   - D1 then restores the prior bytes, so the working copy differs from HEAD, and HEAD holds the new bytes with the trailer.
   - Every re-run of `replace` is then refused by "the working copy equals that committed copy". The named way forward ("re-read the record, rebuild, re-run") never clears that state, so it loops.
   - Evidence: `internal/gitrepo/gitrepo.go:157-166`; compare `add`'s `decisionWayDiffers`, `decision.go:101-103`.

4. **MEDIUM · consistency · D1 bound, `IdentityText`.** `IdentityText` strips every `### Added after Discussion` entry.
   - A replace that rewrites only the body of an added entry, or only drops added entries, leaves `IdentityText` unchanged. A Bouncer re-entry then does not review it.
   - So the stated bound "A replace changes `IdentityText` … reviews it again" is false for exactly the edits D1 newly permits.
   - Evidence: `internal/discussionparser/identity.go:11-19,53-62`.

5. **MEDIUM · design · D1 notice.** The notice tells the agent to "re-read it whole".
   - Merriam (the Webster master) is the role the code says may not read the record; it gets entry text inline instead.
   - A replace notice therefore gives Merriam nothing to act on, and the change never reaches it.
   - Evidence: `internal/loomcli/decision.go:313-316,354-359`.

6. **MEDIUM · design · D4 live-batten refusal.**
   - It names no existing seam a board verb can use. `boardcli` imports nothing from batten (`internal/boardcli/cli.go:16-38`), and liveness reads exist only in `pairteardown` and `loomcli/driverprompt.go:107`. This is a fact left for Plan to find.
   - The check can't be atomic with a batten start. Batten can start after the check, and Record-Write seeds the run from the old stored record before `--record-file` writes, so the run keeps the stale record silently (`internal/loomshed/recordwrite.go:145-157`).
   - It can't see a batten run on the owner machine when the upsert runs on another machine.

7. **MEDIUM · design · D4 "one board write".**
   - "Record file written before `board.json`, as `StoreDecisionRecord` does" has no rollback. For an already-flagged entry, a payload the store's validation refuses after the file write leaves the new record with the old body.
   - That breaks "nothing written" and "the same write as the payload's other fields". `FileRaddle` restores the former record files on a failed write (`raddlefile.go:151-159`); `StoreDecisionRecord` does not (`boardengine/decision.go:57-61`).

8. **MEDIUM · design · D3 board copy.** After a raddle family filing, `convertRunEntry` turns the run's entry into the project and writes the project's own record onto it.
   - D3 copies the run's record onto "the run's board entry when it carries a stored record". An `add` or `replace` on that run after filing in non-perch mode (only perch is refused, `decision.go:942-959`) would therefore overwrite the filed project's record with the Sketch-level run record.
   - Evidence: `internal/boardengine/raddlefile.go:204-228`.

9. **LOW · design · D1 carry-over bound.** The check requires the file to hold every current carry-over block. It never refuses an extra one.
   - A replace can therefore add forged carry-over blocks, which only Go may write. `IdentityText` strips them, so no re-review sees them.
   - Evidence: `internal/discussionparser/identity.go:35-51`.

**Tool calls**
- grep of `writeCarryOver` callers and the fabric commit message flag: to learn which rows write the record outside the decision lock, and whether `lym fabric commit` takes a message that could forge a trailer.
- `internal/shedadapters/bouncercarryover.go` and the `carry_over` config in `shedrecipe`: to confirm the Bouncer seam writes the decision record.
- grep of `carry_over` in the recipes: to confirm Plan-Review and Code-Review Bouncers write carry-over blocks after approval.

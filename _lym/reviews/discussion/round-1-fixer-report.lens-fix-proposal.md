```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. For each finding fixed as proposed, I tried to build a case where the proposal itself is wrong.

**MEDIUM: F1's staged-index refusal has a way forward that loops**
- **Changed text:** D1 commit bullet, "`CommitEmpty` refuses while the weft index holds staged content … `replace` then refuses with the way forward to re-read the record, rebuild the file from it and re-run `replace`."
- **Case:**
  1. A sweep lands the replaced bytes, and the weft index still holds staged content from another path, for example a writer that crashed between `git add` and `git commit`.
  2. `CommitAnchoredPathsWithTrailers` reports committed false, and `CommitEmpty` returns `ErrIndexNotEmpty` (`internal/gitrepo/gitrepo.go:177-182`). Its check is `git diff --cached --quiet` across the whole index, not just the record's path.
  3. The re-run passes the base and working-copy checks, writes the same bytes, gets committed false again, and is refused again. No step in the way forward clears the index.
- **Effect:** the record sits at HEAD with no trailer. D2 then reads the whole history, so F1's premise (`added-decision-dropped`, and `checkDarnRecord` refusing every `add`) stays in force.
- **Sound alternatives:** have the seam commit only the record path with `--allow-empty`, which leaves other staged content alone, or give a way forward that clears the stray staged content.

**MEDIUM: F6 holds back the notice to a Merriam strand of a running run**
- **Changed text:** D1 bullet, "A Merriam strand, which may not read the record, gets no notice … The reason tells the parent to re-enter the run's Webster, so the fresh session carries the replaced record."
- **Case:** a running Webster run with a live Merriam that forks one implementer per batch. A replace now sends it nothing, so it keeps forking implementers on the stale design until the parent acts.
- **Why the reasoning does not carry over:**
  - `heldBackReason` holds a strand back only for a halted run (`internal/loomcli/decision.go:381-390`). Its stated reason is that a halted run's Merriam has no `lym webster run` owning its verbs. That reason does not apply to a running run.
  - `add` reaches a running Merriam with inline text (`decision.go:313-321,354-359`). The proposal is therefore a regression from `add` for running runs.
  - "Re-enter the run's Webster" is no step a parent can take on a running run without stopping it first, and the record names no such stop.
- **Possible fix:** limit the hold-back to halted runs. For a running run, either send an inline summary or name the stop and re-entry steps.

**LOW: F5's refusal "at any row" also covers a family filing's repair**
- **Changed text:** D3, "`add` and `replace` are refused on that run in every mode, at any row."
- **Evidence:** `internal/boardengine/raddlefile.go:445` says "A raddle run's repair of its family filing writes it". So a filed run can go on to repair its filing, and a design call made during that repair would now be refused, with a way forward naming a filed part the call may not concern.
- **Effect:** at most a missed sibling case; I did not trace the repair path further.

No case found against the F2, F3, F4, F7, F8, F9, F11, F12, F13 and F14 proposals:
- **F3:** `add` holds the status lock only briefly inside `ReadJSON`, and I found nothing showing the carry-over settle holds the status lock. I saw no lock-order inversion.
- **F4:** a beam's project record path carries its slug (`ProjectDecisionRecordRel(slug)`, `projectrecord.go:39-57`), so the path in the trailer does tell it apart from the member's discussion record.

Tool calls:
1. grep for `writeCarryOver` callers, `AnchorRel` and `ScopedPathspec`: to check whether the F3 writer is a sole call site and whether F4's paths are distinct. My context lacked both.
2. Read `runRecordOf` and `beamProjectRecord`: to see whether a beam's record path differs from a member's. My context lacked them.
3. Read the `CarryOver` call site and `circlingStatusReader`: to check F3 for a lock-order deadlock. My context lacked them.
4. Read the Bouncer carry-over adapter and grep the shedengine status lock: to see what lock the settle holds when it writes a carry-over. My context lacked both.
5. Read `raddleRunFiling` and grep the filing modes in `raddlefile.go`: to test F5's "every mode, any row". My context lacked both.

```yaml
lens: fix-proposal
outcome: reported
```

I checked the reviewer's proposals the fixer carried out (F1–F14) by building cases where the proposed change goes wrong.

**BLOCKING — D6 slug bullet ("`<N>` is the subject's fix-filing serial, kept beside its holds file, raised by one at each filing and never reset").** F1's premise is still open across a machine switch, which is the feature's own case.
- **Where each piece lives:** the serial sits beside the holds file. That is under `fabricengine.AdoptScratchDir`, which is the Bolt checkout's ephemeral `.lym/adopt` (`internal/fabricengine/junctionnames.go:172-176`), so the serial is per machine. A run's `status.json` sits under `shedrun.RunDir`, which is the durable `_lym` tree (`internal/shedrun/paths.go:121-141`). Adopt's reconcile brings the other machine's run records here (`internal/adoptcli/adopt.go`: "The reconcile may have brought the run records of the other machine's runs").
- **The case:** machine A files `adopt-fix-x-t1`, and it lands, so its done record syncs. The operator then adopts `x` on machine B. B's serial starts at 1, so a red check on B files `adopt-fix-x-t1` again. That meets `doneSlugRefusal` (`dobbyreads.go:291-293`), so the fix task never starts and, as an open front task, blocks the worktree.
- **The same case from the other side:** A's halted `adopt-fix-x-t1` is still on the board when B files `-t1`. That slug is already an entry.
- **Why the reviewer's proposal fails too:** it used the try number, which a hand adopt resets, so it repeats a slug even on one machine.
- **Way out:** keep the serial in a store both machines share (the subject's records or Bolt), or derive `<N>` from the highest `adopt-fix-<subject>-t*` slug found on the board and in the run records.

**MEDIUM — D4 holds paragraph and D5's green-end bullet ("lifts … each `HoldCaller` entry its subject's holds file lists that still stands, then empties the holds file").** Proposal F5 loses a hold to a race.
- **The case:** adopt appends under the holds file's lock. If a second `lym adopt` appends slug S after the green end has read the file but before it empties it, S's hold is never lifted, and the emptying drops S from the list.
- **The result:** D5's rule "each subject whose holds file lists a hold still standing" then never sees S. A run held this way stays held until `lym tender resume`.
- **Evidence:** D5's own bound accepts holds written mid-check, so this window is reachable.
- **Fix:** the green end should remove only the slugs it lifted, in one read-modify-write under the file's lock, rather than empty the file.

**MEDIUM — D5's failed-removal sentence ("a failed removal carries the failed step's error and names `lym board remove <slug>`") and D6's matching notice.** F3's proposal said to drop `lym batten cancel <slug> --remove` only for the nothing-left case; the fix drops it for every case.
- **The case:** when the failed step is the cancel itself (a run record or pair is present and the cancel errs), the entry still carries a run status. `lym board remove` is then the wrong next step until the run is ended. `Rename` refuses "an entry a run holds" (`internal/boardengine/rename.go:24`), which suggests that guard applies to entries with a run status.
- **Fix:** name `lym board remove <slug>` only for the record-drop and entry-removal steps. For a cancel failure, name cancel's own way forward.

**LOW — D8's first paragraph ("D6's pre-filing removal or D5's green-end removal at the subject's next check removes it").** F7's heal depends on a next check that nothing starts.
- **The case:** a filing that dies between create and release also ends that check. Its holds stay, but nothing starts another check until the operator re-runs `lym adopt <subject>`, and no notice tells them to.
- **What would close it:** the reviewer's other option, a dobby-tick release of a draft front task found in `fronttask`'s record, or saying that the operator's re-run is the way forward and where they are told.

Tool calls:
- Read `junctionnames.go` (AdoptScratchDir) and `shedrun/paths.go` (RunDir, StatusFile), to tell which store is machine-local and which is synced; my context showed neither.
- Grepped boardengine for a remove guard on run-held entries; it returned nothing, so the third finding rests on `rename.go:24` alone.

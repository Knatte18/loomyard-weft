```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals carried out as written (F3, F4, F5, F6, F7, F8).

**MEDIUM: F4's latest-ask refusal sends stale findings to a different record.**
- **Changed text:** D4, "The refusal of an `--ask` that is not the latest, while the latest ask is `pending-post` or `open`, names the way forward: answer that ask with `--ask <latest>`." The new D4 criterion repeats it.
- **Case:**
  1. Ask 3 goes out in round k, and the parent's fork starts reviewing that round's record.
  2. The record changes before the fork answers. Either Burler-Fix fixes round k's findings and round k+1 opens ask 4 under D2's confirm-fixes rule, or Discussion-Write rewrites the record after a goto and the history rule opens ask 4.
  3. The fork's answer to ask 3 is refused, and the refusal tells it to answer ask 4.
  4. Re-sent unchanged as `--ask 4`, its findings were written against the earlier record.
  5. D7 forbids the chair to reject or drop a parent finding, and the new coverage entry fails any review that leaves a report id uncovered. So the round must keep findings that may already be fixed or no longer apply, with `BLOCKING` when the item gave no severity.
  6. Ask 4's own request also reached the orch, and the fork handling it is then refused as `done`.
- **Evidence:** D2 (a later round asks after findings, or after a `Discussion-Write` entry); D4 (the answer sets `done` and refuses an ask already `done`); D7 (never moves a parent finding to `## Rejected`; coverage fails on any uncovered id).
- **Repair:** the way forward should say to review the record again for ask `<latest>` before answering it, or leave that ask to the request it came with. It should not say "answer that ask".

No finding against F3, F5, F6, F7 or F8:
- **F6:** the round-1 focus file is written only by the seed. `ensureFocus(1)` rewrites it only when it is absent or does not parse, inside `seedCall`. So comparing the exact write time holds within a seed.
- **F8:** nothing left in the record reads the generation number. The criterion on a new generation's first round rests on the generation bullet that stays.

Tool calls:
- Read `internal/shedadapters/bouncer.go:1615-1660`, plus a grep for focus-path writers, to check F6's premise that only the seed writes the round-1 focus file; my context lacked this code.
- Read `internal/shedadapters/bouncer.go:990-1030` (`ensureFocus`) to confirm it leaves a parsing focus file untouched; my context lacked this code.

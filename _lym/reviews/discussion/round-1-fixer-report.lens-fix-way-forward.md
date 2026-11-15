```yaml
lens: fix-way-forward
outcome: reported
```

Task: check every way forward the fix introduced (refusals, next steps, criteria) for whether it can be reached and leads to progress.

**Findings**

1. **MEDIUM**, D8 step 1: "A hold another machine wrote refuses the adopt, naming the writer machine and the hold's time, and as way forward waiting for that machine's adopt to clear it."
   - Summary: when the writer machine has stopped, the named way forward never makes progress.
   - Evidence: the fix's own D7 Bound says "a hold no live adopt owns, such as one whose adopter stopped, stands until the operator releases it".
   - So waiting is a dead end in exactly the case F3 was raised for, and no test can reach the end of that wait.
   - The refusal states the hold's time but does not say that a hold whose adopter stopped needs the operator. That pointer can be worded without naming `--release`, for example "if that machine's adopt has stopped, ask the operator".

2. **MEDIUM**, D3 offline cancel: "`lym batten cancel` ends the run's processes and strands and commits the intent; the retry finishes the cancel, its finish and any pair removal, once origin answers."
   - Summary: no process is named that owns this "retry", and D2's retry only pushes owed records.
   - Evidence: D2 defines the retry as a run "waiting at that transition with the push owed" that "retries on a backoff". It pushes commits; it never writes a cancel's finish or removes a pair.
   - The cancel verb has exited by then, and its lock "stays local".
   - So nothing carries the cancel from `cancelling` to `cancelled`. Either name the owner, such as the dobby's pass or a re-run of `lym batten cancel <slug>`, or have the offline cancel's output name that re-run as its way forward.

3. **MEDIUM**, D7: "`lym tender resume` lifts no `hold` entry while the Bolt hold stands."
   - Summary: the fix adds a case where `lym tender resume <slug>` does nothing, without saying what it reports or which way forward it names.
   - Evidence: today `tenderengine` refuses a slug that a hub entry covers and names a way forward (`internal/tenderengine/resume.go:109-123`, "way forward: run \"lym tender resume\" to lift the hub pause"). After the fix, the hold covers that slug through a slug entry instead.
   - The record's Constraints require "every refusal the act rule, the holds and D9 add names its way forward and a test reaches it". This one names none, and the only way out, waiting for the hold to clear, needs saying, as in finding 1.

4. **LOW**, D11's new bullet converts every pair "whose slug has a board entry, no batten record and no live process" into a finished cancel with keep.
   - Summary: a pair in that state for another reason would also be reported as kept by a cancel, with no way forward to go on with it as a run.
   - Evidence: the conversion's condition carries no evidence of an actual pre-upgrade cancel. Its only way out is the notice's `lym fabric remove <slug>`, which discards the pair rather than resuming it.
   - An example is a pair whose claim added it but which stopped before the batten record was written.

No further tool calls were made.

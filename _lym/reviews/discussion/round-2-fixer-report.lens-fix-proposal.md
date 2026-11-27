```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out as proposed (F1–F6) and found three faults.

**Proposals challenged:** F1 Land row, F2 one entry per run, F3 hold on every push and the D13 check, F4 owner read, F5 refused state read from the record, F6 merge record written early.

1. **MEDIUM.** D3's new line "a target reads as refused while `fabricengine.RefusedPushStands` finds its record" is wrong for the landing target.
   - **Summary:** that function cannot see a refusal of the prime's code branch. It also reads a refusal of the prime's records push as the landing target's own refusal.
   - **Evidence:** `internal/fabricengine/pusherreads.go:162-178`. For a prime location `codeBranch` stays empty, so only records-side records on the records branch match.
   - **Effect:** a refused landing push never reads as refused, so it is not named as refused in the summary. D5's "the parent's refused-push record is cleared" has nothing in this read to go by.
   - **Needed:** the record should name a per-target read that also covers the prime's code branch.

2. **MEDIUM.** The new D5 owner paragraph says "after an absence it runs only once the dobby's catch-up has run".
   - **Summary:** the catch-up is run-scoped and runs before a run starts. A landing target belongs to no starting run, so after a wake with no new run start nothing runs that catch-up, and the landing push and the `done` writes wait forever.
   - **Evidence:**
     - the Goal (line 14) says "The dobby's catch-up before a run start";
     - `pushOwedForCatchUp(prime, slug)` in `internal/battencli/catchup.go` takes a slug.
   - **Needed:** the record should name which catch-up gates the landing target, or what runs it after a wake.

3. **MEDIUM.** The F1 proposal adds "Batten's Land row … follows the same rule while the parent target owes a parent push: … an unreachable origin at that pull is a `git-transport` stop".
   - **Summary:** the transport rule only applies while a push is owed. Take a landing push that lands at once, after which origin drops before Land pulls: the pull's fetch fails and Land ends Stuck "pull failed".
   - **Evidence:**
     - `internal/battenshed/land.go:265-270`;
     - `internal/fabricengine/pull.go:565-566`;
     - the Goal says "only a refusal that retrying cannot fix stops anything".
   - **Needed:** the transport clause should hold whether or not a push is owed.

4. **LOW.** The new D5 sentence "reads the run as past its merge only when the parent it judges, chosen as the next paragraph says, holds the run's work" points at the wrong paragraph.
   - **Summary:** after this fix the next paragraph is the owner-read paragraph. The past-the-merge parent choice comes two paragraphs later ("While the parent target owes a parent push, the run reader's past-the-merge read reads the local parent…").
   - **Evidence:** the order of the new D5 text.

Tool calls:
- One Bash call read `RefusedPushStands` in `internal/fabricengine/pusherreads.go` to check how F5's proposal works for the landing target, which my context lacked. The same call read `internal/landingshed/finalize.go:1130-1180` to check what reads the merge record for F6, also absent from my context.

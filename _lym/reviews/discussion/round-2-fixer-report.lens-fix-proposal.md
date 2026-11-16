```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as written, looking for concrete cases where the change itself is wrong.

**[BLOCKING] D6 step 2's new sentence on runs not moved (F8) breaks the killed-adopt re-run**
- Changed text: "A run whose owner before step 1 was this machine is no moved run: steps 2 and 3 skip it, and step 4's catch-up pushes its owed commits."
- The case: an adopt is killed after step 1's owner push and before step 3. On the re-run, step 1 finds this machine as the owner already. So steps 2 and 3 skip every run. No takeover entry is ever written, and the old owner never hands over. Local tips are never archived or reset, and a missing pair is never added from origin.
- Evidence: the criterion "D6: an adopt killed after its owner write goes on from tracked state on its re-run and ends as an uninterrupted adopt does", and D6's opening "each step reads tracked state, so a re-run goes on from where the state stands".
- The skip has to key on something else, such as a takeover entry naming this machine already being on origin. "Owner before step 1" is the wrong key.

**[BLOCKING] The new `--forget-former` refusal in D3 (F5) never clears in the case that sends the operator to it**
- Changed text: "`--forget-former` refuses while any owner on the fetched board ... still names that name ... its way forward is `lym adopt` of each named subject first, on the machine whose work it is".
- The case: `--forget-former` is the way forward only when the former name is another registered machine B's current name. Every subject B owns legitimately carries that name as its owner. So the refusal fires while B owns anything.
- The way forward cannot clear it. `lym adopt` on B rewrites B's current name, which is the same name. `lym adopt` on this machine would take B's work.
- This machine's registration stays refused, so it can make no owner write.
- Evidence: D3's "A registration whose former name is another registered machine's current name is refused ... its way forward is `lym machine name --forget-former <name>`".

**[MEDIUM] D3's copied-file refusal (F3) catches a machine that dropped a former name**
- Changed text: "A registration whose id the registry holds under a current name that is neither this machine's current name nor one of its former names is refused as a copied `machine.json`".
- The case:
  1. Machine A (id X) is registered as `a` on hub1 and hub2.
  2. A runs `lym machine name b`. The hub2 sweep fails, so hub2's registry still has X under `a`.
  3. A later runs `--forget-former a`.
  4. A's next registration on hub2 finds X under `a`, which A's file no longer lists. It is refused as a copy.
- The way forward writes a fresh id. A then becomes a stranger to its own entries, owners and holds on every hub.
- Evidence: `machinecli.sweepHub` reports a failed hub and the other hubs go on (`internal/machinecli/set.go:195-215`, `setName` loop).

**[MEDIUM] D4 step 4's push of owed commits (F7) contradicts the owner-machine rule**
- Changed text: "For a run another machine owns ... it first pushes the run's owed commits, records first, when origin's record holds no takeover entry."
- This makes a non-owner push a run's records.
- Evidence: the `PATTERN-owner-machine` change in this record, "Starting, resuming, pushing or landing a run ... only on the entry's owner machine".
- Neither the pattern nor D4 states an exception.

**[MEDIUM] F3 combined with F4: a machine with no config directory can no longer run a dobby**
- Changed text: "A machine registers at its first catch-up (D4) or dobby start after the upgrade", together with "A machine whose `machinename.FilePath` resolves no directory ... its registration is refused".
- Today such a machine runs under the hostname rule: `FilePath` returns ok=false (`internal/machinename/machinename.go:42-51`), and `Name` falls back to the hostname.
- Under the change, every dobby start and catch-up on that machine meets a refused registration. Nothing states what the dobby or catch-up does then: stop, or go on unregistered.
- The F3 proposal made every machine register, which turns F4's refusal from a rare `lym machine name` error into a blocker at dobby start.

**[LOW] D7's orphan first-sight file (F16) goes stale**
- Changed text: "The dobby records each orphan's first-sight time ... written with the notice and dropped with the teardown".
- The case: an orphan gains a board entry or batten record before its grace ends, so the file is never dropped. When it becomes an orphan again later, the old time stands. It gets no new notice and can be torn down at once once idle, against "tears it down no earlier than the orphan grace after that notice".
- The file also needs dropping when the pair stops being an orphan.

**[LOW] D5's upgrade order (F12) leaves a re-run first read writing a second hold**
- Changed text: "The first read pushes the pause intents and the Bolt hold first, and only then drops the `adopt` and `adopt-hold` entries, so a cut first read runs again from the entries left."
- The case: the first read is cut after the hold push and before the drop, and the adopt's green check clears the hold before the next first read. The re-run then finds the legacy entries and writes a fresh hold over a hub already checked green.
- The bullet only skips "another machine's hold". It says nothing about this machine's own hold, or a hold already cleared.

Tool calls:
- grep for `.Held` readers in `internal/`: I needed the existing consumers of `Coverage.Held` to judge F2's proposal. I raised no finding from it, because the remaining reader (`dobbyengine/passes.go:193`) retires with the owner caller.

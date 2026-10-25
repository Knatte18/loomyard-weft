```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: checking that each refusal and next step the fix added can be carried out from the state it names and leads somewhere useful.

Ways forward the fix introduced:
- D7 refusal for a weft with no committed marker → `lyx fabric reconcile`
- D7 dirty-weft refusal → `lyx fabric commit`, or removing the files
- D7 prime hibernate refusal → `lyx hibernate cancel`
- While a move record stands, `lyx adopt`, `Add` from the worktree, and a weft pull or replaying push refuse → wait, then `lyx fabric anchor &lt;slug&gt; --set &lt;B&gt;`
- D10 checkout refusal → a pair of the target branch's own (`lyx fabric add &lt;slug&gt; --adopt-local`), or `--set` and then checkout
- D10 integration conflict → tag the tip, save the changes of the conflicting and later commits, reset to the parent, re-run, write back at `T`
- D7 Not-landed recovery: rename `B/.lyx` back first, refuse when both exist

Findings:

1. MEDIUM. The changed text is D7's dirty-weft refusal: "or an uncommitted marker; way forward: `lyx fabric commit`, or removing the files". The way forward cannot clear a dirty marker.
   Evidence: `lyx fabric commit` stages only the anchored structural dir and the configured pathspec (`internal/fabriccli/weft_verbs.go:191`, `:200`). The weft-root `.lyx-anchor` is outside that, as the record's own plan note says ("not one of today's anchor-scoped `CommitRecordsPaths` calls"). Removing a tracked marker leaves the weft dirty, with the marker now deleted.

2. MEDIUM. The changed text is D10's checkout way forward: "a pair of its own, which wires at the target's anchor, such as `lyx fabric add &lt;slug&gt; --adopt-local`". That claim contradicts D5.
   Evidence: with `--adopt-local`, `Add` adopts a live records branch (`internal/fabricengine/doc.go:673`), so the new pair's weft carries the target's marker. D5 says `Add` takes the new pair's anchor from the fork point, "never from a read of the not-yet-written weft". Under D5 the pair is wired at the parent's anchor over content at the target's. D5 says nothing about an adopted weft, so this way forward leads to a mis-wired pair.

3. MEDIUM. The changed text is D10's conflict way forward, "save aside the changes of the conflicting commit and of every later local commit … re-run", read together with the new D7 dirty-weft refusal. The re-run can loop.
   Evidence: the text says neither where the changes are saved nor that they stay out of the weft, and the reset is described as "keeping the working tree". Changes kept uncommitted under `A/&lt;names&gt;` trip the new dirty-weft refusal on the re-run. Following its `lyx fabric commit` way forward turns them back into a local commit, which the integration replays and which conflicts again. The way forward should say the changes are saved outside the weft.

The other ways forward listed are reachable and make progress.

Tool calls:
- Read `lyx fabric commit`'s help in `internal/fabriccli/weft_verbs.go` to learn what it stages; nothing in my context showed that.
- Grepped `internal/beamcli/dobbydeps.go` for the processes a dobby spawns, to check that the new pass-in-flight wait can end.
- Grepped `internal/fabricengine/doc.go` for `adopt-local` to find the section on adopting a branch.
- Read that section of `internal/fabricengine/doc.go` to see how `Add --adopt-local` treats an existing records branch.

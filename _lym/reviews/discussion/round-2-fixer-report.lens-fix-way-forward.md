```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: I checked that every refusal, error, next step and criterion the fix added can be carried out from the state it names and makes progress.

Ways forward checked:
- Goal: a relative `HOME` or `LOCALAPPDATA` makes the name an error until the variable is made absolute. This can be carried out, and it agrees with D5's refusal to write under a relative variable.
- D3: every `machinename` error names its own way forward, and wrappers add no hostname fix. The wrapper in `internal/battencli/owner.go:132` can drop "fix the hostname" and keep "re-run lym batten run". This can be carried out.
- D3 dobby Bound: the way forward is the one the error names. It can be reached.
- D5: a hand-written `{"name":"osl46"}` plus `lym machine name osl46` appends the hostname rule's name, so `names` changes and the file is written. The sweep's replace set then holds the hostname name. This can be reached and makes progress.
- D5: dropped `names` entries are named in the output. Nothing needs to follow.
- D6 Lost bullet: re-run `lym machine name &lt;name&gt;`. See finding 1.
- D6 committed-but-unpushed: a re-run pushes the stranded commit. `Bolt.PushRecorded` pushes the branch with no condition on a new commit (`internal/fabricengine/bolt.go:141-146`), and `pushOwnerCommit` then checks `HasUnpushed`. See finding 2.
- Acceptance criteria added for D3, D5, D6 and D7 can all be reached from the states they name.

Findings:

1. LOW, D6 per-hub bullet "A hub whose write returns entries in `OwnerWrite.Lost` … reported per entry with … the way forward, a re-run of `lym machine name &lt;name&gt;`, which takes back an entry whose standing owner is a former name".
   - Summary: no way forward is named for a Lost entry whose standing owner is not a former name.
   - Evidence: the replace set selects only owners in the set. A re-run skips such an entry silently and exits zero, so the re-run the line names never addresses it. The line should say such an entry stays the other machine's, and that `lym adopt` takes it back when it should be this machine's.

2. LOW, D6 first recovery bullet "until a re-run of `lym machine name &lt;name&gt;` pushes the commit. The verb's output for that hub names that re-run as the recovery."
   - Summary: on that re-run, the hub's sweep writes nothing and pushes the stranded commit, but the sibling bullet "A hub whose write records nothing is reported as having nothing to sweep" then labels it that way. The operator gets no sign that the recovery took effect.
   - Evidence: `writeOwners` returns no paths when nothing matches (`internal/boardengine/ownerset.go:223-225`). `Recorded` is built only from `written` (`ownerset.go:140-158`), so it is empty after a push-only re-run.

Further tool calls:
- Read `internal/boardengine/ownerset.go:1-140` for `SetOwners`' commit and push path, to see whether a re-run with nothing to write still pushes. My context held only lines 140-290.
- Grepped `Bolt.PushRecorded` in `internal/fabricengine` to check whether it pushes without a new commit. My context lacked that file.

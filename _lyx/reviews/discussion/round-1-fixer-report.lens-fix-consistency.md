```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked whether the changed text agrees with its sibling decisions, criteria and cross-references.

**MEDIUM: D13's exemption collides with batten's notices.** Concerns D13, "A caller with no run, such as hibernate's notify and fabric's push notifier, or one acting from the prime, notifies the prime's orch as today."
- D8 says every batten verb "acts on the prime's location".
- In the code, batten's notices pass that prime location to `NotifyPrime` (`internal/battencli/wire.go:574` and `internal/battencli/drive.go:332`).
- So under "acting from the prime", batten's notices count as exempt and go to the prime's orch.
- That contradicts the sentence before it in D13, which sends batten's notices through the walk "at each delivery".
- It also contradicts the D13 acceptance line that a run in a beam with a live orch reaches `ly:<beam>:orch`.
- The exemption needs to key on "no run's pair" rather than on the acting location.

**MEDIUM: D11's orch refusal has no twin in batten's start guard.** Concerns D11, the third refusal (a live beam orch strand or watcher) and "Batten's start guard applies the first two refusals to a project entry … so a resume cannot skip them".
- `lyx batten run <beam>` can be run directly, or as a resume, now that D8 admits batten verbs from the prime and from beams.
- That path skips the orch refusal, and the landing then tears down a beam whose orch is still live.
- So the claim "a landing leaves no orch strand or watcher behind" fails, and so does the D11 acceptance line saying the same.
- The start guard and Worktree-Create would need to carry the orch refusal too.

**LOW: D11's first refusal offers a way forward that D2 now refuses.** Concerns D11's first refusal, whose way forward includes "moving its `project` off the beam".
- For an open member that has a local pair, D2's new rule refuses exactly that change, naming `lyx batten cancel <slug> --remove` instead.
- That way forward only works for unspawned members, and the text doesn't say so.

**LOW: D1 can point at notes as things to depend on.** Concerns D1's new "the refusal names the project and its members as what to depend on instead".
- D2 lets notes be project members.
- D1 also says "no task may depend on a note", so naming every member can suggest a dependency that is itself refused.
- It should name only the project's task members.

Tool calls:
- I read `internal/battencli/wire.go:565-578` and `internal/battencli/drive.go:325-334` to confirm which location batten's notices pass to `NotifyPrime`. My context had only the `grep` lines, not the location argument in context.

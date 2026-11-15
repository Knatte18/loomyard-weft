```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as proposed (F1, F4–F10, F16, the D3 half of F17, F18, F19 and F20), plus the records-first ordering it took from F2.

1. **MEDIUM**: F1 hand-over of a Finalize past its parent-side merge.
   - Changed text: D1 "`hand_over` when another machine owns the run; its owner finishes even a `landing` past its parent-side merge (D4)", D4 "A run `landing` past its parent-side merge hands over too", D9 "ends here too".
   - Summary: the old owner learns of the takeover only when a push to the run's own records branch is rejected. A Finalize past its merge writes to the parent's branches, which carry no takeover entry, so the old owner keeps writing the parent side while the adopter resumes the same Finalize.
   - Evidence: D4 starts a hand-over on "a run's records push is rejected" and reads only "the run's own record". The parent lock is machine-local (state map 3.2, cited by the review). D9's end can then kill the old owner after it pushed a parent-side merge but before it pushed its records. That leaves parent-side commits that the adopter's resumed Finalize repeats, or local parent-side commits that go under archive tags.

2. **MEDIUM**: F2 records-first ordering versus D8's bound.
   - Changed text: D2 "the records side first, and the code side only once the records side has landed". D4's new bullet "Until origin's code branch holds the tip … the act rule answers `wait` … that machine's next hand-over or catch-up (D6) pushes it".
   - Summary: an old owner that stops between its records push and its code push leaves a transition on origin whose code exists only on that machine. The adopter waits on it forever, with no way forward but that machine running lym again.
   - Evidence: D8's bound says "adopt works from pushed state alone and never asks about another machine's local state". The new wait makes the adopter depend on another machine's local state, and nothing in the record lets the operator drop the stranded transition.

3. **MEDIUM**: F5's notice names a way forward that is refused when several machines upgrade.
   - Changed text: D7 "That first read posts one notice … and a re-run of `lym adopt` on that machine as the way forward".
   - Summary: every machine that ever adopted holds standing adopt entries, so each machine's first read after the upgrade writes its own Bolt `adopt` hold. Each machine's re-run of `lym adopt` is then refused over the other machine's hold, and the named way forward cannot be carried out.
   - Evidence: the review's F5 says "every machine that has adopted holds them". D8 step 1 says "A hold another machine wrote refuses the adopt". Only `--release` clears both holds, and the notice may not name it.

4. **MEDIUM**: F8's `pauserecord.Admit` pass-through, with D2's retry.
   - Changed text: D2 "A transition's push passes the pause record's network gate, as `pauserecord.Admit` lets a lock holder through".
   - Summary: an admission makes every network call of the process pass the gate, on any goroutine and in any repository. It also stays marked in flight. Two consequences follow:
     - An owed push that retries on D2's backoff with origin unreachable keeps an in-flight marker under a pause or hibernate request for as long as origin is down, and hibernate's watch waits on those markers.
     - In a process that serves several slugs, a pause of another slug is bypassed for as long as the admission stands.
   - Evidence: the `Admit` doc in `internal/pauserecord/hold.go` says "the pass-through is process-wide for the length of the admission … a call on another goroutine or in another repository passes too". `internal/hibernateengine/watch.go` reads the in-flight markers. D2 says an owed push "retries on a backoff".

5. **MEDIUM**: F16's teardown condition strands landed runs that the upgrade paused.
   - Changed text: D11 "with no pause or hold covering it and no other machine owning it".
   - Summary: D7's upgrade turns every standing `adopt` entry of an owned run into a run pause intent, landed runs included, since past adopts leave those entries. A landed run with a pause intent is never torn down. It is neither kept nor an orphan, so nothing reports it.
   - Evidence: D7 says "a standing `adopt` or `adopt-hold` entry of a run this machine owns becomes a run pause intent (D3)", and nothing excludes terminal runs. D11 lists and notices only kept pairs and orphans.

6. **MEDIUM**: F19's `orchcli.BeamUp` after a release that stops the adopt before steps 2 and 3.
   - Changed text: D8 "An adopt that stops on a release still runs `orchcli.BeamUp` for the subjects it already owns".
   - Summary: released after step 1, the adopter owns the subjects but has neither reset its stale local tips nor written takeover entries. Its dobby then acts on stale branches.
     - D6 step 3 pushes only local commits that sit ahead of origin and never fast-forwards a branch that is behind. The fixer put F10's fast-forward in D8 only, not in D6.
     - A rejected push finds no takeover entry, so D4 replays the stale run over the old owner's newer transitions.
   - Evidence: D8 step 1 writes the owner, steps 2 and 3 do the reset and the takeover, and step 3 holds the fast-forward. D4's third bullet: "A rejection whose fetched record holds no such entry … replays and pushes as today."

7. **MEDIUM**: F7's stop rule reads a session record that the old owner can still rewrite.
   - Changed text: D10 "When the ref names another machine that the session record names as the orch's last launcher or resumer, it stops and writes nothing", and the restore rule "When the session record names this machine, the local transcript wins".
   - Summary: the session record is committed and pushed at every orch launch and resume, and no takeover entry guards it, because takeover entries sit only in batten and child records. Two cases follow:
     - An old owner whose orch starts before it reads the takeover pushes a session record naming itself. The new owner's next start then restores the old owner's stale mirror, which is F7's original failure.
     - The new owner's mirror, whose lease is lost to that mirror, stops and writes nothing.
   - Evidence: D10's first bullet says "The orch start commits and pushes it at each launch and resume". D4 writes takeover entries only "into each moved run's batten record and child record". No decision gates an orch start on the act rule.

Further tool calls:
- One grep of `internal/` for in-flight marker readers in `pauserecord` and `hibernateengine`. My context lacked whether hibernate waits on the network in-flight markers that `Admit` sets, which finding 4 needs.

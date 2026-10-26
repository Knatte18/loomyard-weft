```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (fix-proposal lens).

**Findings**

1. **BLOCKING**: D9's landing-hold bullet: "its run status reads as no run hold to every board and batten reader" (the F1 proposal).
   - Summary: F1's rule covers every reader, so it also covers readers that ask whether a run is in flight at all, not only whether a landing hold stands. Those readers stop seeing a pawl run parked at Beam-Await.
   - Evidence: D5 keeps "adopt's test of a started entry, which reads the run status". Under the new rule a pawl run at Beam-Await reads as not started to adopt, so the F5 fix ("resumes a beam's pawl run whenever … this machine owns both the beam and its parent") never finds the run to resume.
   - Evidence: D5 also names "tender's owner-pause selection" as a run-status reader, and D9's owner-yield bullet relies on "the batten pass's owner check owner-pauses the run". Both are board or batten readers that the blanket rule now tells there is no run.
   - Concrete case: G is parked at Beam-Await on Neve, and OSL-1046 runs `lyx adopt root` and then `lyx adopt G`. Adopt's started-entry test reads `running · Beam-Await` as no run, so it neither resumes the run nor reports it as owned elsewhere.
   - The rule needs to cover landing-hold readers only: membership, the Project layer, the run lock, the fork precondition, `ResolveBattenPrime`, D3 and peers. In-flight readers should still see a run: adopt's started test, tender's selection, batten's owner check and dobby's driven-run set.

2. **MEDIUM**: D6: "It skips them for a beam whose pawl run holds the landing hold (D9), reports that beam naming `lyx batten status <slug>`, and still writes its owner." (the F6 proposal, first alternative).
   - Summary: writing the owner of a beam whose landing is in flight on another machine can strand the landing.
   - Concrete case: G is in Parent-Align on Neve, root is owned by Neve, and OSL-1046 runs `lyx adopt G`.
     1. The owner write moves G to OSL-1046.
     2. D11 ("A pawl run's owner verdict reads the beam project's own owner") owner-pauses the run at Neve's next batten pass.
     3. D6's new resume clause does not resume it on OSL-1046, because OSL-1046 does not own G's parent.
   - Result: no machine drives the landing. The reported way forward, `lyx batten status <slug>`, only shows the pause; it does not name `lyx adopt root` or anything else that resumes the run.
   - The review's other alternative avoids this: refuse to adopt a beam that holds the landing hold, naming `lyx batten status <slug>`. Failing that, the report should name `lyx adopt <parent>`.

3. **LOW**: D9 accepted-request bullet: "When they fail it restarts the beam's dobby" (the F4 proposal).
   - Summary: on a failed re-check this restarts a dobby the operator may have stopped on purpose.
   - Case: the operator runs `lyx beam stop G` and then `lyx beam land G` while a member is open. Beam-Await stops the dobby (already stopped), the check fails, and Beam-Await restarts a dobby that was not live before the request.
   - D4 bounds a stopped dobby as restarted only by "a hand `lyx batten run` there".
   - The restart should happen only when Beam-Await itself stopped the dobby.

4. **LOW**: D9 Finalize bullet: "the halt names `lyx batten cancel <member> --remove` for each, then `lyx batten run <slug>`" (follows from the F4 alternative).
   - Summary: in the window between Beam-Await's re-check and its status write, a plain `lyx board upsert` can add an unstarted member, not only a hand `lyx batten run`. For such an entry the named cancel has no run to cancel.
   - Evidence: `BeamLandRefusal` refuses on "any open entry's `hubgeom.ParentOf` is the beam" whether or not that entry has a run. The bullet frames the case only as "a member a hand `lyx batten run` started".
   - The halt should also name moving the entry off the project, or marking it done, for a member with no run.

Tool calls: none.

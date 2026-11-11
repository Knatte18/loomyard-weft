```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that each way forward the fix introduced can be carried out and leads somewhere that makes progress.

Ways forward the fix introduced:
1. D4: when the origin probe fails, the upsert is refused, with the way forward "re-run it once origin answers".
2. D4 acceptance: the refusal bullet now lists "an upsert whose origin probe fails".
3. D6: a fourth parent-directive point says to "record each folded call again with `lym loom decision add`" before sending a run back to a Webster row after a fold. The new Open risks line and Review bullet say the same.
4. D2 acceptance: a new bullet for the empty-trailer variant ("HEAD already holds X from a sweep, a replace lands an empty trailer commit…").
5. D3 acceptance: `add` on a beam run leaves the project entry's stored record unchanged.

Findings:

- LOW. Changed text: the D6 fourth point and the new Open risks line, which ends "D6's parent directive has the parent record the call again with `add` first".
  Summary: this only helps when the parent itself sends the run back to Webster. If a human runs the `lym loom goto`, the folded call is never re-added, and the Open risks line reads as if the risk were fully covered.
  Evidence: the tapestry recipe header says Drift-Verify and Webster Stuck "halts the whole run" for a human, and the review's F1 calls the goto a fix through Webster. That goto may be an operator's. `contracts/stencils/parent/parent-directive-parent.md` has no line mentioning `goto`, so nothing tells the operator about this step. The Open risks line should say the re-add happens only when the parent makes the goto, and that an operator goto after a fold leaves the call out of Master's view.

The rest checked out:
- Way forward 1: re-running once origin answers can be done and makes progress. The test row can be reached by pointing origin at a path that does not answer.
- Way forward 4: the scenario can be reached. With HEAD holding X, a replace with `--base-sha256` set to X's hash and file X passes the subset check, because X's added entries equal the current ones. Under D1 it then lands an empty trailer commit.
- Way forward 5: the scenario can be reached on a pawl run.

Tool calls:
- One Bash call: read the tapestry recipe header around the Webster and Drift-Verify Stuck text, and grepped the parent directive for `goto`. My context did not have the recipe's escalation text or the directive's contents.

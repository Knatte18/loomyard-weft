```yaml
lens: fix-way-forward
outcome: reported
```

No findings list was asked for; four findings.

1. **MEDIUM.** Concerns D1's commit bullet: "`CommitEmpty` refuses while the weft index holds staged content … `replace` then refuses with the way forward to re-read the record, rebuild the file from it and re-run `replace`." The same rule appears in D2's seed sentence ("lands an empty trailer commit as D1 does").
   - **Summary:** the way forward cannot make progress. Nothing it names clears the staged content, so every re-run reaches the same refusal. A seed that hits it fails the gate on every resume.
   - **Evidence:** `gitrepo.Repo.CommitEmpty` refuses on any staged content in the whole index, not only the record's path (`internal/gitrepo/gitrepo.go:177-182`). After this refusal HEAD holds the written bytes without a trailer. A rebuilt file passes the base check, `CommitAnchoredPathsWithTrailers` again reports committed false, and `CommitEmpty` again returns `ErrIndexNotEmpty`. D1 also forbids `lym fabric commit`, which would land the staged content.
   - **Further effect:** while this lasts, D2 reads the history whole, so `checkDarnRecord` refuses every later `add` with `added-decision-dropped`. Its way forward ("restore the entry and commit the record") undoes the replace.
   - **Needed:** the refusal has to name a step that clears or lands the staged content.

2. **MEDIUM.** Concerns D4's new worktree refusal ("way forward names `lym loom decision replace &lt;slug&gt;` in that worktree, or … `lym batten cancel &lt;slug&gt;` first") together with D3's new filed refusal.
   - **Summary:** for a family project whose raddle run has filed, D4's first way forward leads straight into D3's refusal.
   - **Evidence:** `RaddleFile` is the arras recipe's last row (`internal/loomrecipe/arrasrecipe_test.go:61` has `{NameRaddleFile, "", …}`). The filed run stands at that row, and its task worktree stays in the hub. So `--record-file` on the project slug is refused by D4's worktree check, which names `replace` in that worktree. D3 now refuses `add` and `replace` there "in every mode, at any row". D3's own way forward points at the filed parts, which never carry a call that belongs to the project record.
   - **Needed:** only `batten cancel` remains, and D4 does not say it applies to a run that has already finished filing. One of the two refusals has to name the way out for this case.

3. **MEDIUM.** Concerns D1's Merriam bullet: "The reason tells the parent to re-enter the run's Webster, so the fresh session carries the replaced record." The matching D1 criterion has the same problem.
   - **Summary:** the step names no verb, and on a running run it cannot be carried out from the state it names.
   - **Evidence:** the existing `heldBackReason` fires only for a halted run, where the re-entry step is `lym webster run` (`internal/websterengine/doc.go:509`). The new bullet holds Merriam back on a running run too, where a live `lym webster run` owns the session. The record names no way to end or restart that run so a re-entry can start, and it does not say whether a fresh Master prompt carries the record.
   - **Needed:** the reason should name the verb, and say what to do first while the run is running.

4. **LOW.** Concerns D1's commit-error bullet: "When HEAD holds the written bytes without that trailer, it lands the empty trailer commit as the bullet above says."
   - **Summary:** nothing says what happens when that fallback commit fails. If its error is the staged-index refusal, the run is back in finding 1.
   - **Needed:** state the fallback's own refusal and its way forward. No criterion reaches this case.

Outside my lens, one claim belongs to fix-proposal. Making the `board` seed trailer cut the history means a stale re-seed (Open risk 3) no longer raises `added-decision-dropped` for entries `add` recorded after the first seed. That silences the one signal of that risk.

Tool calls:
- One grep of `internal/websterengine` and `internal/loomcli` for the re-entry verb, and of the recipe for the RaddleFile row's successor. My context lacked both: what re-entering Webster means, and whether a filed raddle run keeps standing in its worktree.

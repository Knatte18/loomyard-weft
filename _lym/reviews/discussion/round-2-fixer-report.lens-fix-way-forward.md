```yaml
lens: fix-way-forward
outcome: reported
```

Fix-way-forward lens: checking that each way forward this fix introduced can be carried out from the state it names and makes progress.

**MEDIUM: the Land restore for the not-live sentinel fails when the weft branch still exists locally**
- **Changed text:** `decision-record.md` D2, the not-live sub-bullet at Land, together with D1 step 3's new reuse of a standing records worktree.
- **Summary:** The restore names `git -C &lt;records repo&gt; branch &lt;weft branch&gt; &lt;tag&gt;` even when the sentinel came from a task branch that is gone locally and on origin while the weft branch is live. That `git branch` fails because the branch already exists.
- **Evidence:** D2's closed set names the sentinel "for a weft that is not live or a task branch gone locally and on origin". Before this fix, `add.go:328-331` refused a standing records worktree before the probes ran. With the new reuse, a pair whose records worktree stands on its local weft branch, but whose task branch is gone, now reaches the warp probe's not-live sentinel. The bullet then fills in a `git branch` over a branch that exists and is checked out in that worktree.
- **What the bullet should say:** name the tag fetch and the weft `git branch` only when the weft is not live, and name only the task-branch restore otherwise.

**MEDIUM: a records branch checked out in another records worktree still fails pass after pass**
- **Changed text:** `decision-record.md` D1 step 3, the sentence "The `AdoptLocal` refusal of a records branch checked out in another records worktree does not count that reused worktree as another", and D2's closed set.
- **Summary:** That refusal stays in force for any other records worktree, but D2's closed set gained only its twin for the task branch. The records-branch refusal is therefore transient, and dobby retries it every pass while the status stays `running`. That is the outcome the Goal rules out, and no way forward is named for it.
- **Evidence:** `internal/fabricengine/addadopt.go:44` returns the plain `fmt.Errorf` "--adopt-local: records branch %q is checked out in the records worktree %s …". D2 classes every refusal outside the closed set as transient.

**LOW: the checked-out refusal's own text points the operator away from batten**
- **Changed text:** D2's "On the task branch checked out in another worktree: the refusal's own way forward, `lym fabric checkout &lt;other branch&gt;` in that worktree".
- **Summary:** D2 says the stop's error carries `Add`'s refusal text, and that text ends "then re-run `lym fabric add &lt;slug&gt; --adopt-local`". So the stop names a hand `lym fabric add` next to `lym batten run &lt;slug&gt;`. A hand add takes the plain path, without the adopt-only option.
- **Evidence:** `internal/fabricengine/addadopt.go:27`.

The other ways forward hold:
- **`cancel --remove` before Land:** `Remove` finishes a pair whose task worktree is gone and archives and deletes the weft on origin (`remove.go` doc above `Remove`). It returns `ErrPairNotFound` only when nothing is left, which `removePairAndBranch` tolerates. That clears both the not-live and the `ErrRemoteLeftover` parts.
- **The stray-path delete:** `Remove` never deletes a stray path (StrayPath), so a hand delete is the right step.
- **The `--unset-upstream` step:** it follows a leased delete that already dropped the tracking ref, so it runs cleanly.
- **The cleanup-window note:** it leads back to a restore that still works from the surviving tag.

Tool calls:
- `remove.go` lines 130-175 and its `ErrPairNotFound` lines: my context lacked what `Remove` does with an absent task worktree, which decides whether `cancel --remove` clears the parts `Add` refused on.

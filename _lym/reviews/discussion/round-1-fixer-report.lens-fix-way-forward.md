```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, the refusals, next steps and rows the fix introduced in `_lym/discussion/decision-record.md`.

Ways forward the fix introduced:
1. D2: a `--single-repo` marker push refused as protected → "allow direct pushes … then run the clone again".
2. D2: a `--single-repo` marker push refused for another reason → "pushes nothing further" (no way forward named).
3. D2: a one-URL clone refuses when the prime lacks `.lym-anchor` → names the missing marker (no way forward named).
4. D5: step 3 refuses when a restored path changed after step 2 → "as today's dirty refusal does".
5. D5: a merge whose source edited `.lym-anchor` refuses → names the marker change (no way forward named).
6. D5: bringing a branch level refuses on an unpushed non-records commit (no way forward named).
7. D7: an owed `deferred` prime push → "`git push`".
8. D7: a `protected` prime push → "allow direct pushes …, after which the next records push … carries the records".
9. D7: a records commit off the prime's recorded branch → "switch the prime back to the recorded branch".
10. D8: a publish against a base other than `parent_branch` refuses, naming both (no way forward named).

Findings:

- **MEDIUM**: D7, "Its notice names the operator's unpushed commits and `git push` as the way forward, and a run in the prime waits at its transition until the operator pushes", together with "Once such a record exists for the prime's branch, the guard's refusal is recorded as `protected` too."
  F3's premise is still open for a branch that is protected and has operator commits unpushed. Only a records push that reaches origin can learn of the protection. The guard refuses before any push while operator commits are unpushed, so no `protected` record ever forms. The refusal is recorded as `deferred`, which `FailedPushesOwedBy` holds (`internal/fabricengine/failedpush.go:272-293`). Its named way forward, `git push`, is refused by the protection, so the prime run waits for good.
  This also covers a one-URL clone of an existing hub, which never probes, and protection added after clone.
- **MEDIUM**: D2, "The one-URL clone of an origin holding `lym-bolt` refuses when the prime's branch lacks `.lym-anchor` at its root, naming the missing marker."
  The refusal names no way forward, and the state it names has none reachable through lym. A rerun with `--single-repo` refuses because origin holds `lym-bolt` (D2 Bound). A plain rerun refuses again. Nothing in the record commits the marker on origin's prime branch. This goes against `PATTERN-refusal-way-forward`, which the review's F3 cites.
- **LOW**: D2, "way forward: allow direct pushes to that branch …, then run the clone again", and "pushes nothing further when that push is refused".
  The rerun meets `hub already exists at &lt;hubPath&gt;` (`internal/fabricengine/clone.go:162-163`, `:232-233`) unless the refused clone removes the hub directory it made, or the rerun passes `--reset`. The record says neither. The non-protected refusal names no way forward at all.
- **LOW**: D5, "A merge whose source changed `.lym-anchor` other than through fabric's anchor-change commit refuses, naming the marker change", and D8, "the verb refuses when that branch differs from the branch's `parent_branch` …, naming both".
  Neither refusal names a way forward. For D5 that could be reverting the marker edit on the source branch. For D8 it could be retargeting the pull request or correcting the origin record. As written, a run stopped there has no stated step to make progress.
- **LOW**: D7, "Fabric refuses to commit records in the prime while it is not on that branch …; way forward: switch the prime back".
  The record never says the transition commit is held rather than lost while the prime is on the other branch. Live records written meanwhile are uncommitted tracked edits, and `git switch` can refuse them when they differ between the two branches. The way forward may then fail, with no step named.

Tool calls:
- `grep` of `internal/fabricengine/clone*.go` and `internal/fabriccli/clone.go` for existing-target refusals: my context did not show how clone treats an existing hub directory on a rerun.
- `sed -n` of `internal/fabricengine/clone.go:150-170,220-240`: to read the exact "hub already exists" refusal the D2 rerun meets.

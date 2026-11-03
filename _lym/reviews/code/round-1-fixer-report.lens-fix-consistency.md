```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked the change set against its surroundings. Four cross-references and twins turned up, none of them a defect in the change:

- **`createRefusal` and its caller:** the new parameter reaches both call sites (`wire.go:987` and the one test call), and the doc comment and table rows agree with it. `wire.go:337` words the parameter clearly.
- **Twin of the F4 fix:** `readoptstop.go:155` words `ErrTaskPathTaken` as "stands where the task worktree goes; save anything worth keeping from it, then delete it".
  - It does not check `PairWorktreeStands`, but it never claims the path is "not a worktree".
  - It was outside F4's `location`, and the rubric excludes improvements the review never asked for. I note it only in case you want the twin made consistent.
- **Doc cross-references to `PairWorktreeStands`:** `fabricengine/doc.go:613` and `boardcli/doc.go:26` describe it as "registered linked worktree" and make no claim about listing failures. Both still hold.
- **`isRegisteredLinkedWorktree` comment:** its "failure to enumerate answers false" is about that helper only. The new unlistable→true rule is documented on `PairWorktreeStands` itself, so the two do not contradict.

Tool calls made (2):
1. `sed` on `fabricengine/doc.go`, `boardcli/doc.go`, plus a grep over battencli, to read the doc cross-references to `PairWorktreeStands` and look for twin wording. My context lacked those files' text.
2. `Read` of `readoptstop.go:135-165` to see the twin's `ErrTaskPathTaken` handling. My context lacked that file.

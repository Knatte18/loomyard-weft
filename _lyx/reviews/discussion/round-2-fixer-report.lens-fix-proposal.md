```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: testing the proposals the fixer carried out (F1–F10) against concrete cases where they break.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F10. F2, F3, F4, F5, F9 and F10 held up: I could build no case that breaks them. F2 relies on adopt lifting the owner pause, which `internal/adoptcli/doc.go:42` confirms it does.

**MEDIUM — F6 makes every offline pass a full pass, so Open-Window re-runs on every probe tick.**
- Changed text, D4: "dobby keeps the full pass owed until a full pass reaches origin", together with the `--continue-pass` bound "with no full pass owed".
- Case: origin is down for an hour and no reed client is attached to the task's session (a headless machine, or the operator closed the window). Every pass is then a full pass. Each one runs `reopenWindow`, which runs `batten.open_window_command` whenever no client is attached (`internal/battencli/arm.go:724-757`). That opens a window once every `notice_probe_s`, where it used to open one per wake.
- Each pass also retries the origin reconcile over the network, which lengthens the pass.
- The proposal needs the Open-Window re-run limited to one per owed episode, or a full pass with origin unreachable should skip it.

**MEDIUM — F7's refusal closes only one of the ways to get a member owned here under a beam owned elsewhere.**
- Changed text, D2: "`lyx adopt` of a member task whose parent beam ... reads owned by another machine refuses ..."
- Case: machine B adopts the beam away from A. `PATTERN-owner-machine` says "Adopting a beam moves the beam only". So a member with `owner_machine` A recorded stays owned by A while its parent beam is now B's.
- A's dobby for that beam exits, because D1 says "A dobby that reads its beam owned by another machine ... exits". B's dobby drives only runs on B's machine (D3). The member's run on A therefore still has no dobby.
- The review's premise, "the adopted run has no dobby", still holds on this path. The refusal-only option leaves it open.

**LOW — F8's release wording is ambiguous and, as written, wrong for merges.**
- Changed text, D5: "An upsert whose payload sets `status` to null, and a merge whose `set_status` is nil, are releases".
- A merge with no `set_status` at all also reads as "nil": `MergeTasks` skips the step when `setStatus == nil` (`internal/boardengine/store.go:1074`). Only `MergeStatusUpdate.Status == nil` clears the status (`store.go:438`). The sentence should say "a merge whose `set_status` step sets a null status".
- The same paragraph has a second conflict. A creating upsert whose payload sets `status: null` matches both "creates a task with no status ... give it `draft`" and "sets `status` to null ... are releases". D5 doesn't say which rule wins.

**LOW — F1 says dobby reacts to adopt's first pass, but dobby can't see that pass's exit.**
- Changed text, D4: "until its owner pause is lifted or `lyx adopt` takes the run back, each of which runs a full pass (D4's lift trigger, and adopt's own first pass); it drives the run again only when such a full pass exits 0".
- Adopt's first pass is started detached through `ResumeInWindow`'s replacement, not by dobby, so dobby never reads its exit code.
- Recovery in fact comes from the owner-pause lift that adopt makes in step 4 (`internal/adoptcli/doc.go:42`), which fires dobby's lift trigger.
- If `PauseOwnedElsewhere` had failed, there is no owner entry to lift. Dobby would then never pass the run again, and the run would sit marked owned elsewhere after the adopt.

Tool calls:
- grep of `internal/adoptcli` for its pause-lift and resume steps, plus a search for the Open-Window re-run code: my context lacked adopt's step order and where Open-Window re-runs.
- `sed -n 715,770p internal/battencli/arm.go`: to read what `reopenWindow` does on each full pass, which my context lacked.

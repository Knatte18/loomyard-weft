```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix for F1 (task-branch removal) and F2 (stale done-run docs).

Findings:

- **MEDIUM — `internal/beamcli/cli.go:786`.** The `lym beam create` help still says "run is still refused, naming `lym batten cleanup --apply`". It is the same claim F2 fixed in `internal/beamcli/doc.go:59` and `docs/overview.md:816`. The F2 fix covered those two sites and missed this twin. It is the user-facing help of the verb the other two describe, and it now says the refusal is unconditional.
- **LOW — `internal/battencli/arm.go:857` (`doneSlugRefusal`).** The refusal tells the user to delete the leftover task branch by hand. It says nothing about the dobby's done-run clear, which now removes the run and its branch once the landing is counted and the notice decided. F1's fix text offered this edit only if the branch were kept, so it is not required. The text is still accurate for the window before the clear. A reader would not learn that the dobby normally does it.
- **LOW — `internal/dobbyengine/doc.go:485`.** "A landing whose run records `lym batten cleanup --apply` removed before dobby read them is not counted." This still holds, since the dobby's own clear waits for the count. It is not stale.
- **No finding — `internal/battencli/cli.go:316`.** It names the "leftover branches" in the batten help. It describes the refusal and is unchanged by the fix. I did not check it further.
- **No finding — `clearDoneRun` callers.** There are two production call sites, `ClearMotifRun` (nil discard) and `ClearDoneRun` (discard wired). Both are updated. The only other callers are the two test sites, both updated to the new signature.

Unchecked: whether `docs/overview.md:1750` ("The task branch stays on the remote…") describes the pair teardown or a done run. I did not read its context.

Tool calls:
1. One `grep` across the Go and Markdown sources for `batten cleanup --apply`, task-branch-stays wording, and `ClearDoneRun`/`clearDoneRun(` sites. Context lacked a repo-wide sweep.

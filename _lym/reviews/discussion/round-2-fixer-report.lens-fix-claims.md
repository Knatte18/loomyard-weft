```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

1. **MEDIUM.** Changed text: D4 step 3, "A refusal is every error the cancel raises before it ends anything: the hub-pause refusal, the owned-elsewhere refusal, …, an origin-stopped refusal and a nothing-left refusal."
   Summary: the list leaves out errors the cancel raises before it ends anything, and those errors read the same as the partial-finish text. The output therefore cannot settle the "before it ends anything" test.
   Evidence:
   - `judgeOrigin` raises its errors before it ends anything: "origin reconcile … failed …; way forward: fix the cause it names, then run … again" (`internal/battencli/cancel.go:612`) and "pulling the board … failed …" (`cancel.go:620`).
   - The same holds for `probeRefusal` (`cancel.go:301`, `:308`, `:328`) and the beam-cancel refusal (`cancel.go:515`).
   - `cancelRun` says it "refuses before it ends or removes anything, unless a step's error says otherwise" (`cancel.go:254`). The output does not show whether processes ended.

2. **LOW.** Changed text: D4 step 3, "an origin-stopped refusal".
   Summary: no refusal in the cancel matches this name. The origin-unreachable error comes after the cancel has ended the run's processes, so the fix's own definition makes it a partial finish.
   Evidence: `cancel.go:248`, "origin was unreachable, so the cancel ended the run's processes and stopped; way forward: run … again once origin answers".

3. **LOW.** Changed text: D2 Rationale, "each writer may write only its own output files".
   Summary: the evidence covers only the Discussion writer and the Webster master. The plan writer, which D2 also names, has no matching limit in its template.
   Evidence: a grep of `contracts/stencils/loom/loom-template-plan.md` for "may write", "only files", "read-only" and "never write" finds only line 47, which concerns operator additions.

4. **LOW.** Changed text: D4 step 3, "pushes the code tag by hand with the command a notice would name, `git -C <prime> push origin refs/tags/archive/<slug>/<12 hex>`".
   Summary: the notice names `git -C <job.Worktree>`, not the prime. The two commands differ, though the tag is shared through the repository.
   Evidence: `internal/fabricengine/remoteteardown.go:187`.

These claims checked out:
- The hub-pause refusal text (`cancel.go:304`).
- The owned-elsewhere refusal (`cancel.go:715`).
- "landed by another machine" (`cancel.go:658`) and "reached its parent" (`cancel.go:331`).
- A slugless `tender resume` lifts every entry (`tendercli/cli.go:194`).
- The upsert refusal while the records branch reads live (`recordupsert.go:38-46`).
- The board refusing a `project` held by a run (`boardengine/project.go:99`).
- The ready record at `_lym/landing/beam-ready.json` (`loomengine/config.go:106-114`).
- The code store keeping default refspecs (`PATTERN-single-repo-hub.md:29`).

Tool calls:
- grep of `cancel.go` for its way-forward and refusal sites: my context held only `cancel.go:290-314` and `:700-724`.
- `boardengine/project.go:85-115` and a listing of the plan stencils: neither the `project` refusal path nor the plan writer's limits were in my context.
- A grep of `loom-template-plan.md` plus `cancel.go:240-262` and `:314-332`: I needed the plan writer's limits and the order of cancel's steps.
- `remoteteardown.go:270-335`, grepped for push: I needed the notice's push command, and this returned nothing useful.
- A grep of `remoteteardown.go` and `taskbranchdiscard.go` for `git -C` and `refs/tags`: this found the exact command the notice names.

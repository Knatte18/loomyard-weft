```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F7 change set against its surroundings.

1. **MEDIUM.** Changed text: `contracts/orchrole/orch-role.md`, § Notices, the new paragraph after the list.
   The section still says "Only these reach you:" before a list that omits the raddle waiting line, and the new paragraph then says that line "reaches you too".
   The same section opens by defining a notice as text starting `[batten notice]`, `[dobby notice]` or `[run watch]`, and says "Handle each under Investigating a stop".
   Evidence: the unchanged lines "Each mail body or line whose text starts with … is a notice. … Only these reach you:" against the added "A line opening `raddle: …` reaches you too. It is no stop".
   The two statements contradict each other. Moving the line into the exclusive set, or rewording "Only these", would settle it.

2. **LOW.** Changed text: F5's autonomy sentence in `internal/dobbyengine/template.yaml` and `internal/beamsetting/beamsetting.go` ("a severe halt, a merge-ready beam or a raddle split's approval").
   Three twins still say "only for a severe halt or a merge-ready beam":
   - `internal/dobbyengine/config.go:33`
   - `internal/dobbyengine/tasksetting.go:50`
   - `internal/heddleengine/glossary.md:27`, the glossary's `autonomy` entry

   Evidence: a grep for `merge-ready beam|severe halt or` over `internal`, `contracts`, `docs` and `cmd`, excluding tests and `role-autonomy`. A phrase wrapped across lines could escape it.

I found no other disagreement in the F1, F2, F3, F6 and F7 changes. `docs/overview.md` around line 1978 still names arras among what `Run-Shed` seeds, which remains true of the code.

Tool calls (2 of 6):
- Grep for the autonomy phrase twins: my context held only a head-truncated search that missed `config.go`, `tasksetting.go` and the heddle glossary.
- Grep of `loom-rubric-raddle-review.md` and of the overview and package docs for recipe-list and `land_notice` wording: my context did not hold the rubric or those doc lines.

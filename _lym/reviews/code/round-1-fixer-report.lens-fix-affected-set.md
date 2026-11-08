```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 fix covers every site of the unqualified "never change the recipe" wording.

No findings.

Unchecked list: none. F1 is the only finding, and it is a one-site wording premise.

- Ran a grep across `plugins` and `contracts` for "change the recipe", "raddle filed" and "recipe of a task" in `*.md`. My context lacked any search outside `board-policy/SKILL.md`. The only hit is the fixed line, `plugins/ly/skills/board-policy/SKILL.md:48`, so no twin with the unqualified ban remains.

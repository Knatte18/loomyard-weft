```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over commit 84306fb5.

List: the diff changes no function, branch or error path. It edits only prose in `plugins/ly/skills/board-policy/SKILL.md`.
- New bullet at `SKILL.md:44-48`: no executable code, so there is nothing to run. Not applicable.
- Reworded split bullet at `SKILL.md:84-85`: no executable code. Not applicable.

Findings: none. Wording and scope questions belong to the other lenses and the handler.

Tool calls: none. The context already held the diff and file.

```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List, per behaviour the diff adds or changes, the test that pins it.

Per-behaviour list:
- New bullet in `plugins/ly/skills/board-policy/SKILL.md:44-48` (a tapestry-sized note is proposed as a `motif` task; raddle splits the Sketch; raddle's recipe choice stands). Test: none. This is static skill prose, which the lens exempts.
- Reworded Sketch-split bullet at `SKILL.md:84-85` (a Sketch is split by promoting it to its raddle task, never by hand). Test: none. This is static prose and also exempt.

The diff changes one markdown skill file. It changes no code, no behaviour and no test, so no behaviour is left uncovered. I found no findings.

Tool calls: none. The diff and file contents were already in my context.

```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The change set agrees with its surroundings:
- **Skill text:** the new `SKILL.md` paragraph matches the `promote` help text, which says to give such a task's dependencies in `depends_on` because the dobby can start the released task before a later `set-deps` lands.
- **Tests:** the new rows use the existing helpers and fixtures. The refusal text they assert matches the `promoteTo` error. The notice line matches `siblingNotice`'s format string. The map key alignment is gofmt-clean (`lym loom fmt` reported no change).
- **Other references to `board promote`:** the remaining hits (`adoptcli`, `boardengine/sketch.go`, `boardengine/task.go`, `docs/overview.md`) only mention the verb in refusal messages or prose. None describes the payload shape, so none went stale.

Tool calls:
1. `grep -rln "board promote"` over `.md` and `.go` files: to find twin references to the promote payload that the changed text might need to agree with. My context did not hold them.
2. `grep -n "board promote"` in `docs/overview.md`, `internal/adoptcli/adopt.go`, `internal/adoptcli/doc.go`, `internal/boardengine/sketch.go` and `internal/boardengine/task.go`: to read how those hits use the verb. My context lacked the surrounding lines.

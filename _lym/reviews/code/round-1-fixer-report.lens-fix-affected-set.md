```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the review findings and check that the fix covers every member.

**Findings left unchecked:** F4 (I ran no search of its own) and F7 (its premise is not a set). F1, F2, F3, F5 and F6 were searched.

1. **MEDIUM**: F5, `internal/orchengine/prompt.go:54` (`autonomyOnLine`, not changed)
   - The line appended to the orch's pointer while autonomy is on still says "reach the operator only for a severe halt or a merge-ready beam, as role-autonomy.md beside your role says".
   - The orch receives this text. It contradicts `role-autonomy.md:7` and the updated `template.yaml:7` and `beamsetting.go:17`, which add a raddle split's approval.

2. **LOW**: F5, twins of the changed autonomy and land_notice comments left stale:
   - `internal/heddleengine/glossary.md:27`: the **autonomy** definition still says "only for a severe halt or a merge-ready beam".
   - `internal/dobbyengine/config.go:30-33`: `LandNoticeKey` lacks the motif-line sentence added to its `beamsetting.go` twin, and `AutonomyKey` lacks the split approval.
   - `internal/dobbyengine/tasksetting.go:50` (`LookupAutonomy`): "It reaches the operator only for a severe halt or a merge-ready beam."

F1, F2, F3 and F6 have no uncovered members in what I searched:
- **F1:** the batch path is the one that projects the incoming entry before it validates.
- **F2:** I found no other Go message or config comment that offers arras. The `arras:` keys that remain in `runprofile/template.yaml` are kept on purpose.
- **F3:** the new line covers the operator-addressed notes in Steps 3 and 4.
- **F6:** the remaining `arras` mentions I saw in `docs/overview.md` describe code that still runs.

Tool calls:
1. Grep of board write sites: my context lacked the full set of recipe-writing paths.
2. Grep for `arras` and recipe-name users in `internal`, `cmd` and `contracts`: the earlier search covered only some directories.
3. Grep of the raddle stencil, rubric and spec for operator-addressed notes: my context held no search of the spec.
4. Grep for the autonomy and land_notice wording repo-wide: the earlier search was cut off by `head -30`.
5. Grep for `arras` in docs and plugins: the earlier search covered only two doc files. The output was truncated to a preview.
6. Read of `fileRaddleEntry` and the batch callers, a second doc grep, and the stale `prompt.go`, `config.go` and `tasksetting.go` lines: my context lacked them.

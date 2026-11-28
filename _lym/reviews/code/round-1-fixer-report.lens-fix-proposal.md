```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals (F1, F3) that the fixer carried out.

**Findings**

1. **MEDIUM**: the new settling sentence in `darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`, against the unchanged closed-scope way forward a few lines below it.
   - **Summary:** a note filed exactly as the stop report says can fail to settle the stop, so the resumed writer stops again.
   - **New trigger:** the note must be titled `Decision for <slug>:`, "says the scope is closed" and "files the measurements for a new task".
   - **Stop report's instruction:** it still says only "name instead a board note filing the measurements for a new task, then `lym loom resume`". It gives neither the title nor the closed-scope statement.
   - **Concrete case:** the operator files a note titled, say, "Measure test times" that files the measurements, then resumes. The writer reads only notes titled `Decision for {{.slug}}:` (`darn-template-write.md:50`), so none settles the point and it stops again. This is the loop F1 reported.
   - **Siblings:** `darn-template-write.md:62` says "a board note saying the scope is closed". `darnStopWayForward` (`internal/loomshed/darnwrite.go:32`) names both the title and the closed-scope statement. The halt Reason carries that constant after the report, which reduces the risk, but the report's own way forward is still incomplete.
   - **Change:** have both fragments' line name a note titled `Decision for <slug>: <call>` that says the scope is closed and files the measurements.

F3's replacement ("the resumed writer runs them under that entry"): no case found against it.

I did not establish what the handoff does when a measurements-only entry is settled this way and the branch holds no commit. `darnwrite.go` showed no empty-branch check in my grep, but I did not read the later gates, so I raise no finding on it.

**Tool calls**
- One `grep` over `internal/loomshed/darnwrite.go` and `internal/loomrecipe/scopeline.go`, for how an empty handoff is treated. My context held neither file's body.

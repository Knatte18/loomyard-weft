```yaml
lens: fix-code-correctness
outcome: reported
```

Task: checking the correctness of the F1/F3 fix (fix-code-correctness lens).

**Findings**

1. **LOW**
   - **Changed text:** the new closed-scope exception, in both `darn-fragment-make-change.md` and `darn-fragment-make-change-gated.md`, read against the unchanged line "on a task a Code-Review round has run on, name instead a board note filing the measurements for a new task, then `lym loom resume`."
   - **Summary:** the resumed writer now accepts only a note that meets conditions the stop report's way forward does not ask for.
   - **Evidence:**
     - The exception requires a note titled `Decision for <slug>:` that "says the scope is closed and files the measurements for a new task".
     - The way-forward line asks only for "a board note filing the measurements", with no title and no scope-closed statement. A note written to that wording alone does not settle the point, so the writer stops again.
     - Step 3's twin (`darn-template-write.md:62`) at least says "a board note saying the scope is closed".
     - The harm is limited: `internal/loomshed/darnwrite.go` appends `darnStopWayForward` to the halt Reason after the report, and that text names both the title and the scope-closed statement.

**Traced with no finding**

- **Closed scope, no parent, note present:** `Call` removes both stop markers and respawns the writer. The writer finds no allowing entry but finds the note, so it runs no calls, creates no marker and hands off. `Call` then takes the `Done` path, which ends the resume loop.
- **No note:** the writer stops as before.
- **An allowing `Added after Discussion` entry:** it is still checked first.
- **Tests:** the fix added none. The lines the existing assertions match are unchanged.

**Tool calls**

- `sed -n 120,300p internal/loomshed/darnwrite.go`: read `darnWrite.Call`'s resume and stop-marker handling. My context held only grep hits and the review's summary of it.

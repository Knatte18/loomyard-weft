```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 and F2 fixes cover every site in their sets.

**Findings left unchecked:** none. I reran the searches for both F1 and F2.

**Findings**

- **LOW**, `_lym/landing/summary.md` lines 3-4, which F1's rewrite of the line below them reaches.
  - Summary: "It is now named `lym: <shortname>`, such as `lym: odd`" still states the name for every form.
  - Evidence: the sentence the fix added says "the Windows desktop shows the file name". `doc.go:21` says the same, and `hubPath` builds that name as `lym-hub-<shortname>.cmd`.
  - The lead sentence is therefore unqualified in the way F1 faulted `doc.go`. The qualification follows two lines later, so the harm is small.

**Rest of both sets**

- **F1:** my search of tracked files outside `docs/research` found no other claim about how a launcher displays its name. The matches at `doc.go:17-21` and `desktoplauncher.go:170` ("titled", which the review judged accurate) agree with the code.
- **F2:** `desktoplauncher.go:138` was the only comment blurring which line carries the title. The `Name=` and `rem` rows in `desktoplauncher_test.go` agree with the new wording.

**Tool calls**

- `git grep` for launcher show/name/title claims across tracked files, excluding `docs/research`: my context held only the package files and the summary, not other possible claim sites.
- `git grep` for title, displayed-name and comment-line wording in `internal/desktoplauncher` and the dobby launcher files: my context did not hold the test file or the dobby launcher files.

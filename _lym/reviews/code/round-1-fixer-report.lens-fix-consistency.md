```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- `ViewProject`'s only constructors are the root literal and the `add` closure in `viewProjects`. The root literal leaves `Done` false, which is correct, and every `add` call now passes the new argument.
- The only consumers of `state.view.projects` in `board.html` are `buildTree` (line 403), `projectView` and `projectSelect` (line 589). `buildTree` and `projectView` do not read `done`, so a done project still shows in the tree under its parent. The select now skips it, so the create and edit forms no longer offer it.
- The `BoardView.Projects` doc comment (view.go:41-42) still matches the code. The `viewProjects` doc comment is unchanged and still accurate.
- The `viewFixture` doc comment now matches its five projects: P1, P2 and P4 open, P3 and P5 done, P5 nested in P3.
- The new P5 row cannot disturb the other fixture tests. It has no entries, so `EntriesStandInTheirProjectAndSection`, the section list and `len(view.Entries)` are unaffected, and the gate passed on tier0.
- Only `view_test.go` builds `ViewProject` values; the grep found no docs or other tests that list the projection's keys.

Calls made:
1. Grep for `design_file`, `ViewProject` and `projects` across the repo, to find twin consumers or constructors of the projection that my context did not show.
2. Read `board.html` lines 398-448, to check how `buildTree` and `projectView` handle the project list, since my context only held the `projectSelect` span.

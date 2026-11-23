```yaml
lens: fix-proposal
outcome: reported
```

No findings for either finding the fixer fixed as proposed, F1 or F2. I did not review F3, which went to the parent.

**F1 (the `done` flag and the dropdown filter)**
- Both forms start from a value the filtered options still hold. The edit form starts at `KEEP_PROJECT` (`board.html:607`) and the create form at `''` (`board.html:612`), so `select.value = form.values.project` always finds an option.
- No other use of `state.view.projects` offers a done project as a write target. The only other reads are `buildTree` (`board.html:403-409`) and the tree view, which still need done projects to nest their members.
- `Done` is correct for every row `viewProjects` builds. The open walk passes `false`. The done loop passes `true`, and `isDone` is the same test that loop already filtered on.
- The JSON key `done` is new and collides with no existing `ViewProject` key.

**F2 (the nested done project in the fixture)**
- The new project P5 sits under done project gamma and stands as a project, not an entry. It adds no row to the entry map, so the `T1`–`T11` placement map and its entry-count check are unchanged.
- Its expected row `Parent: "P3"` is not the `projectHandle` root fallback, so the mutant that hard-codes the root handle fails the test.
- The expected order of P3 before P5 depends on `RenderOrder` listing done projects in fixture order. The gate passed with that order, so it holds.
- P4's expected `Parent: "P0"` is unaffected, because `childProjects` keeps only open projects. The gate passing agrees.

Calls made: none. My context already held every file I needed.

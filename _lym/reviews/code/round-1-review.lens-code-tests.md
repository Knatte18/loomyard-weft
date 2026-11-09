```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour**
1. A null status on an empty-status project releases the project and its direct draft members, in store order. This is pinned by `TestSetStatusReleasingProject_ReleasesDraftMembers` (`projectrelease_test.go:44`). It asserts the released slugs `[d1 d2]` and every other slug's status.
2. Released, run-held and done members, nested projects with their members, and outsiders stay untouched. This is pinned by the same test: the `released`, `running`, `landed`, `nested`, `nested-member` and `outsider` rows.
3. A failing member refuses the whole write, with the project and the other members unchanged. This is pinned by `TestSetStatusReleasingProject_Refusals/MemberFailingValidation` (`projectrelease_test.go:98`).
   - The failing `bad` member comes after `d1` in store order.
   - The row asserts that `d1` is still `draft` and that the project status is unchanged, so the unsaved-mutation guarantee is covered.
   - It also asserts the error names the project, `member "bad"` and the dangling dependency.
4. A project with a run status or `done` is refused, naming each member's release as the way forward. This is pinned by the `RunStatusProject` and `DoneProject` rows (`projectrelease_test.go:104`, `:109`). Each asserts the release command line and that nothing was written.
5. A project with a hand-set status takes a plain clear and releases no member. This is pinned by `TestSetStatusReleasingProject_OtherWritesReleaseNoMember/ProjectWithAHandSetStatus` (`projectrelease_test.go:154`).
6. `Board.SetStatus` and `UpsertTask` release no member. This is pinned by the `SetStatus` and `UpsertTask` rows of the same test (`projectrelease_test.go:149`, `:150`).
7. The set-status verb calls the new method and its notice names the released members. This is pinned by `stepProjectRelease` (`verbs_test.go:555`), which asserts the exact notice and that the member's status is cleared.
8. A non-null status, or any non-project entry, passes through to `Store.SetStatus` unchanged. **None.** See F1.
9. The refusal text for a run-status or `done` project with no draft member ("has no draft member task to release, so nothing was written"). **None.** See F2.
10. The refusal names each member when there are several. Only single-member rows exist, so the `"; "` join is unpinned. See F2.
11. A selector that matches no entry gives `task not found`. **None.** This is a minor gap; I raise it only inside F1.
12. Merge, batch and batten cancel's claim clear release no member. Merge and batten cancel have no row. These paths are unchanged by the diff, so I raise no finding. The `Board.SetStatus` row covers the shared null-status path.
13. The help text and doc changes are prose and exempt.

**Findings**

- F1, LOW, design, `internal/boardengine/projectrelease_test.go:138`. No row calls `SetStatusReleasingProject` with a non-null status, or with a task or note that has a null status, or with an unknown selector. The decision record requires the old behaviour for those inputs. The test's doc comment says "A plain status write", but the `SetStatus` row exercises `Board.SetStatus`, not the new method. If the guard `status != nil || current.Kind != KindProject` (`projectrelease.go:44`) were dropped, no test would fail. Evidence: `projectrelease_test.go:149`, `projectrelease.go:44`. Fix: add rows that call `SetStatusReleasingProject` with a non-null status on a project with a draft member, and with a null status on a task, asserting the member stays `draft`.

- F2, LOW, design, `internal/boardengine/projectrelease_test.go:97`. The `projectReleaseRefusal` branches are only partly reached. The empty-members branch has no row, and the multi-member join has no row. Evidence: `projectrelease.go:76-85`; every refusal row uses the single member `d1`. Fix: add a `DoneProject` row with no draft member, and give one run-status row two draft members whose release commands are both asserted.

- F3, LOW, consistency, `internal/boardcli/verbs_test.go:550`. `stepProjectRelease` was inserted between the doc comment of `stepProjects` and its function. The lines "A member's project change is refused while a worktree stands…" now sit above `stepProjectRelease`'s own comment, and `stepProjects` has no doc comment. Evidence: the diff hunk at `verbs_test.go:550-567`. Fix: move the new function and its comment above the orphaned comment block, so those lines attach to `stepProjects` again.

**Tool calls:** none. My context already held every file I needed.

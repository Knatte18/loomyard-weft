```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked what the diff's changes force outside the lines they touch. I found nothing wrong, so I raise no findings.

**Per change, the set found and whether it is covered**

| Change | Set found | Covered? |
|---|---|---|
| `PullRequestDeps.ParentReviewWanted` and `AskParent` | The constructors are `loomcli/landingdeps.go:347`, `envkit.go:221`, `prreview_test.go:640` and `finalize_pr_test.go:471`. The two seam-listing tests are `battencli/wire_seams_test.go` and `loomcli/wiring_seams_test.go`. | Covered. The first three are filled. The seam tests list both fields as loom-only or run.go-assembled. `finalize_pr_test.go` leaves them nil, and `askParentReview` reads nil as off. `checkWiring` correctly does not require them. |
| Config key `pr_review_ask_parent` | `config.go` (struct and `ConfigSpec` category `none`), `template.yaml`, `config_test.go`, the `docs/overview.md` key list (line 1329), and the key-classification test. It is a new key, so no reinterpretation is needed. `configreg_test.go:146` lists the local-closed keys; this key is open to the local layer, so it does not belong there. | Covered. |
| `Record.ParentReviewAsked` (`parent_review_asked`, omitempty) | `pullrequest/record.go`, `askParentReview` (the writer and reader) and `assertAsks` in the tests. No `.md` file describes the record shape. The field is omitempty, so older readers and writers tolerate it. | Covered. |
| `[pr-review request]` mail line | The sender is `<shortname>:LOOM`, via `orchcli.NotifyParent` with `loomProgram`. The orch-role text (`orch-role.md:191`) names the same prefix and sender. No mail-prefix table or test lists the other `[...]` prefixes (`role-slack.md` does so only for Slack). | Covered. |
| Reviewer stencil `landing-template-pr-review.md` | The existing rules are intact: the `gh` reads and comments at lines 41-44, "never set the body with gh pr edit", and waiting for the operator's next message. The new section only adds the parent-review flow, with `lym creel post --parent --expect-reply`. That matches `PATTERN-stencil-ownership`'s "the agent asks its parent". | Covered. No rule is dropped or weakened. |
| `orch-role.md` new section | It tells the orch to read the diff with `gh pr diff` through a read-only fork. The orch role is the one embedded exception and is fixed in `contracts/orchrole`, as `PATTERN-stencil-ownership` allows. It says "Never approve, merge, close or comment on the pull request", which fits `PATTERN-github-auth`. I found no deny rule for `gh` in the settings I searched, so there is no `PATTERN-refusal-way-forward` conflict. | Covered. |
| `docs/overview.md` and `landingshed/doc.go` | The overview adds a PR-Review row note and the key note. `doc.go` describes the ask: when it happens, once per record, the prefix, and what it names. Both match `prreview.go`. | Covered. |
| Declared PATTERN changes | The directive I inherited shows none. The diff touches no `PATTERN.md` or `pattern/` file. | Nothing to land. |
| Commit messages | Neither commit has a `Pattern-Departure:` trailer. | No departure to judge. |

**Notes outside my lens, for the handler.** These touch the answer key, so they are not affected-set findings.
- The board entry says the run "waits at PR-Review until the orch answers satisfied or sends findings". `doc.go` says "The row's endings do not wait on the orch's answer." The code only asks once and then runs the existing wait, so nothing makes the run wait for the orch.
- The stencil says the orch's answer arrives "as creel mail" to the session. The request tells the orch to answer the session's strand, so that part is consistent.

**Tool calls (2 of 6):**
1. Searched for other lists of loom keys, other `PullRequestDeps{` constructors, docs describing the record, `gh` mentions in the contracts and orch engines, and orch mail prefixes. My context lacked these repo-wide sets.
2. Searched the three key-list tests for `code_auto_accept` (the ones that would need the new key), the orch-role mail lines, and overview PR-Review mentions. This confirmed no other table needs an entry.

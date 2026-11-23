```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. Two LOW findings, no MEDIUM or BLOCKING.

**LOW — `docs/overview.md:954` still makes the claim F1 narrowed elsewhere.**
- Changed text it concerns: the F1 fix to the `toggle` long help, `doc.go:30-31` and `summary.md`.
- Summary: the overview sentence says a failed click shows "on the bar and in the trace log" with no qualifier.
- Evidence: the fix narrowed the trace-log claim in three places. A click from a directory that is no lym worktree logs nowhere, but `docs/overview.md:954` still reads "a click that fails shows why on the bar and in the trace log."
- `_lym/landing/summary.md` says the change updates `docs/overview.md`, so it is part of the change set.
- Suggested edit: "…shows why on the bar, and in the trace log when the click runs in a lym worktree."

**LOW — the `reportToggle` doc comment has the same unqualified claim.**
- Changed text: `internal/slackcli/toggle.go:150-152` says "the bar and the trace log are the only places a failed click shows."
- Summary: this implies a trace log is always written, which the F1 fix says it is not.
- Evidence: the F1 fix narrowed the long help and the package doc but not this comment, which sits in the same file.
- Suggested edit: "the bar, and the trace log in a lym worktree, are the only places…".

Checked and consistent:
- `summary.md` "Tests" lists the `TestToggle` refusal subtest and the `-d 10000` expectation. It does not list the two new subtests, which is an omission only, since it is not an enumerated list.
- The `summary.md` line saying no test pins the Warn line was reworded to match the new test.
- The `slacktoggle.go` file header ("a Slack toggle's click asks for") is still accurate for the callers.
- The doc.go line "A failed toggle is therefore shown …, its pre-run's refusals included, and logged at Warn" now reads consistently with the next sentence.

Calls made:
1. `grep -rn "trace log"` over `internal/slackcli`, `internal/reedengine`, `internal/orchcli`, `docs/overview.md` and `README.md`, to find twins of the narrowed claim that my context lacked.

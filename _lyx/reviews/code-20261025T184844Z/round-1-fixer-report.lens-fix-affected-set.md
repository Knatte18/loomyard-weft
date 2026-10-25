```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F4 cover every member of their sets.

Findings:

- **LOW — F2, stale "overview window" naming in `internal/orchengine/doc.go:13`.** This line is not changed text. It already says "the session's overview window named `beam: prime`, or `beam: <slug>` in a beam", so it agrees with the new `docs/overview.md` row. No action needed.
- **LOW — F2 twin, `internal/loomcli/bootstrap.go:25` and `internal/orchcli/runs.go:36`.** Both comments still call the strand "the session's overview window", with the same wording as the phrase the review flagged. Neither names the beam title. In `runs.go:36` the title is a parameter, so the comment holds. These are not stale, and I raise them only because they are siblings of the fixed phrase.
- **No finding — F1.** `atOrBelow` has one caller (`overview.go:382`), and that caller already logs the error at Debug. The cycle-guard test row covers `loop-x` and `loop-y`, and both stay out of the beam `a` window's list.
- **No finding — F4.** Searching for "its own first" in Go and markdown finds no remaining claim about the window's own orch coming first, apart from unrelated uses of the phrase.
- **Unchecked — F3.** The only set is `dobbyStatus` run cells. I did not search for other callers of `DobbyLines` that lack a cells test.

Tool calls:
1. One `grep` over `*.go` and `*.md` for `atOrBelow(`, "its own first", "overview window" and "overview, agents". My context lacked a repo-wide view of these sibling sites.

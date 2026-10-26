```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the fix-affected-set lens, that the F1, F4 and F7 fixes cover every member of their sets.

No findings.

- **F7 (verb-list comments):** a repo-wide grep for the `recover-batch … fix-verify` list form, excluding lines that already name `fix-plan`, found one remaining site. `internal/websterengine/runlevel.go:777` says Master backgrounds `recover-batch` and `fix-verify`, and the wait marker leaves them out. That comment is not one of the agent-spawning or pause-covered verb lists the finding names, so I didn't count it.
- **F1 and F4 (stencil tables):** every Go file that names `webster-prefix-verify-fix-cold` is now matched for the plan-fix stencil too. `roleOpeningStencils`, the marker table and the friction table all carry it, and `render.go`'s header already lists it. The `docs/research/…` report is a dated research document, out of scope.
- **Run-state sites (beyond the review):** the other places that loop over `st.VerifyFixers` (`strand.go:157`, `runlevel.go:328`) also handle `st.PlanFixers` (`strand.go:162`, `runlevel.go:339`). `runwrites.go:138` omits `PlanFixers` on purpose, per D7.

Tool calls (3):
1. A grep for verb-list comment sites without `fix-plan`: my context held only the edited sites, not a repo-wide search.
2. A grep for the stencil name `webster-prefix-verify-fix-cold` and for the plan-fix stencil name: needed to compare the tables keyed by stencil.
3. A grep for `VerifyFixers` and `verify-fixer` usages, then `PlanFixers` in the sibling sites (two calls): needed to confirm the plan fixer is covered wherever the verify fixer is.

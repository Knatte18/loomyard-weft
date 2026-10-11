---
round: 3
focus:
  - "internal/clihelp/index_test.go, the t.Run loop at the end of TestRenderIndex_FormsAndFiltering, and internal/clihelp/index.go visibleLocalFlags (which calls cmd.LocalNonPersistentFlags): settle whether any subtests still run in parallel over one shared cobra command tree, so that lazy flag merging can write pflag.FlagSet maps from several goroutines, and whether the test's coverage of RenderIndex forms and audience filtering is unchanged."
---

The Publish verify failed with `concurrent map writes` in `internal/clihelp`; the focus entry targets that failure only.

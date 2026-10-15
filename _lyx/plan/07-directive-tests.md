# Card 7 — Prune the directive and orch pointer tests

**Edit:**
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/parentdirective#TestDirective_EmptyNameRendersNoParentVariant`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `internal/pattern#TestDirective_PerRole`
- `internal/orchengine#TestRenderPointers_OneLineNamingTheirPaths`

**Intent:**
Prune the directive renderers' tests by D1, keeping filled values, branch anchors and the banner, marker and blank-line checks:

- `TestDirective_RendersEveryValueVerbatim`: every want shrinks to the filled value it carries ("lyx check --now", "STEP-CHECK", "STEP-FORMAT", "REJECTION-RULE-SENTENCE", "the PLACE section", "EXPLORE-STEP", "INPUT-RECORD", "LAST-FILE", "FILES-DIR", "GOVERNING-INPUT", "DISPUTE-EVIDENCE", "`/abs/one.md` and `/abs/two.md`", "SPAWN-POINT-SENTENCE", "SEVERITY-RULE-PHRASE", "/abs/rubric.md", "lyx check --final" and the lens list), dropping the prose around each; the shared wants "forks are outstanding is allowed.\n4. Judge every finding" and "returned no report.\n6. Run `lyx check --final` again" go.
  The branch anchors "died before its Agent call" (relaunch part) and "Spawn one fork, fork0" (writer-fork part) stay, and so do the wantNots; the inOrder list becomes "SPAWN-POINT-SENTENCE", "Spawn one fork, fork0".
- `TestDirective_EmptyNameRendersNoParentVariant`: drop the "SendMessage" absence check; "No parent is recorded" tells the variant apart.
- `TestDirective_RendersTheRuleWithoutBannerOrMarkers`: keep "Edit or Write", the anchor other tests find the edit directive by, and drop the other eight phrases.
- `TestDirective_PerRole` in `internal/pattern`: drop the "under pattern/ " half of the background-pointer check and the "_lyx/" absence check; keep the `patternDirName + "/"` check, the overview, the heading and banner checks and the equality with the filled embedded default.
- `TestRenderPointers_OneLineNamingTheirPaths`: drop "none is re-armed" and "DEFER" from the wants and the "watcher"/"re-arming" absence check; keep the one-line check and the path wants.

**ImpactSummary:** test-only across `internal/twilldirective`, `internal/parentdirective`, `internal/editdirective`, `internal/pattern` and `internal/orchengine`; no production code changes.

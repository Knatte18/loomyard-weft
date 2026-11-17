---
round: 2
focus:
  - "internal/heddleengine/target.go, internal/shuttleengine/claudeengine and internal/shuttleengine/codexengine: is each engine name and each instruction file name (the claude local file and the codex AGENTS.override.md) now declared once as a named constant, with no second literal added by the diff?"
  - "internal/shuttleengine/codexengine/manifest.go setupCandidates and internal/shuttleengine/claudeengine/manifest.go: does an omitted plugin hook with an empty plugin name land as the same omitted plugin-context item, keyed by hook name, in both engines' records and verdicts?"
  - "internal/shuttleparity/prepare.go ReferenceEngine and ComparedEngine and their uses in internal/shuttlecli/manifest.go: is the engine-name conflict with PATTERN-code-conventions settled by derivation or by a Pattern-Departure trailer on a commit that names the entry and gives a reason?"
  - "internal/shuttleparity/report_test.go, internal/shuttlecli/manifest_test.go, cmd/lym and internal/shuttleengine/claudeengine tests: do the added tests pin temporary-directory removal on error paths, the unrun and stripped-verdict output fields, the manifestReportOf wiring and no-shortname refusal, the production passage reader, and the Claude omitted-hook, failed-resolution and no-repository branches, without duplicating an existing test?"
  - "internal/burlerengine/review.go, internal/landingshed/fixreview.go and internal/shedadapters/bouncer.go: do both the seed producer and the judge producer render the same prompt text as before the diff, with RenderJudgePrompt traced from judgeCall?"
---

Round 1's judge ratified the one focus departure, that the renderers live in `internal/shedadapters/bouncer.go`, and asks round 2 to close the judge-producer trace it left open.

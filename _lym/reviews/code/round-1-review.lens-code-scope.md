```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, tracing each change in the diff to a goal point of the decision record.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries. The `## Goal` Idea has two points:
- **G1:** Burler writes the round's diff as one file per package, each under the Read cap.
- **G2:** Burler names those files in the explore step, through `internal/burlerengine/prompt.go` and `contracts/stencils/burler/burler-step-1-explore.md`.

**Per change:**
- **C1: splitter and its write in Review.** `internal/burlerengine/diffpieces.go` `writeDiffPieces`, `groupDiffByPackage`, `boundedUnits`, `packUnits`, `pieceFileName`, and `review.go:126-129`. Traces to G1. `diffPieceMaxBytes` = 60,000 bytes stays under the Read cap of about 25,000 tokens (`diffpieces.go:141-144`).
- **C2: the `Profile.Diff` field and the unexported `diffPieces`** (`profile.go:56-62`). Forced by C1, since burler needs the diff text as input.
- **C3: listing the pieces in the explore step.** This covers the new stencil `burler-diff-pieces.md`, the `diff_pieces` marker in `burler-step-1-explore.md`, the registry entry in `stencils.go`, and `diffPiecesBlock` and the optional marker in `composeReviewPrompt` (`prompt.go:139-152`, `518-534`). Traces to G2.
- **C4: the assistant reviewer gets the same pieces.** There is no code change: `assistProfile` copies `*p` along with `diffPieces`. Only the `doc.go` line covers it. Forced by C1 and C3, because the assistant composes from the same profile.
- **C5: the producer reads the diff once per spawning call.** `BurlerHalfDeps.Diff`, `BurlerReviewProducer.diff` and `readDiff` (`burlerreview.go:79-81`, `373`, `457-469`). Forced by G1. The goal keeps burler writing the pieces, and burler is git-free, so the text has to come from the producer.
- **C6: the `stage_diff` row key and `subjectDiff`.** `entries_burler.go:61-69` and `102-118`, plus the `requireSeam` refusal. Forced by G1 as the opt-in that reaches the code subject's patch. It reuses the existing `Env.CodeSubject` seam and adds no `Env` field, which fits the PATTERN-shed-recipe-registry setting-lookup rule.
- **C7: `stage_diff: true` on the tapestry, darn and sampler `Code-Burler-Review` rows, and on the `tapestry-rows-before-motif.yaml` fixture.** Forced by G1, because without these rows the feature stays off for the Code-Review reviewer named in the Seen section. The fixture is forced by the tapestry row.
- **C8: package docs.** The `doc.go` paragraphs of burlerengine, shedadapters and shedrecipe. Forced by C1, C5 and C6.
- **C9: tests.** `diffpieces_test.go`, the `burlerreview_test.go` diff rows, the `entries_burlerhalves_test.go` `stage_diff` subtests, the `template_test.go` and `prompt_test.go` optional-marker rows, and the `markers_test.go` row. Forced by C1, C3, C5 and C6.

**Per decision:**
- **G1** is built by C1, C2, C5, C6 and C7.
- **G2** is built by C3, with C4 extending it to the assistant.

**Findings:** none. Every change traces to G1 or G2 or is forced by one that does. Both goal points are built, and no commit carries a `Pattern-Departure:` trailer.

Tool calls: none.

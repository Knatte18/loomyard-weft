# Fixer-report, code review round 1

F1 is `decision`-class and went to the parent, so this report leaves it out.

## Changes

- F2: `RenderOverview`'s comment now says the derived entries come from the Bolt checkout's agents.yaml alone and that an invalid one is noted.
  `overviewText` writes a `Note:` line with `DerivedWarning` under the claude-settings section.
  `TestRenderOverviewSaysWhenTheEffortDefaultsAreLeftOut` pins that the render succeeds, the note shows and no derived entry shows.
  Fixed as proposed, except that no argument is added to `overviewText`, since `Layers` already carries `DerivedWarning`.
  Of the two ways the review gave to deal with the machine-layer agents.yaml, it took the documented one and did not resolve dirs the way `Render` does.
- F3: `CommitsOwnOverviewNeverLands` in `replayer_test.go` passes a stale `HEDDLE.md` beside a `heddle.json` change.
  It asserts the overview on disk equals a fresh render of origin's merged store.
- F4: `TestRenderOverview` gained a backtick-run entry (four-backtick fence) and a non-object claude-settings entry (shown as written in a json block).
  `TestRenderClaudeSettings` gained `RepositoryEntryWinsOverTheDerivedEffort`.
  The overview rows landed in the F2 commit, since both findings edit `overview_test.go`.
- F5: the registration comment in `cmd/lym/main.go` says the replayer also claims `HEDDLE.md` and renders it afresh after each replay.
- F6: the blank line before `## Lym instructions in code` is back.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `TestRenderOverview`'s doc comment was stale. Fixed as proposed.
- fix-consistency, LOW: the F5 comment line was over 120 characters. Fixed as proposed, by splitting it at the clause.
- fix-proposal, LOW: same F5 line length. A duplicate of the finding above, fixed with it.
- fix-proposal, LOW: the note format `Note: %s.` could double a period. Fixed by dropping the format's trailing period.
- fix-proposal, LOW: the F2 and F4 changes share a commit. Deferred, see below.

The self-check changes are in one commit, `5f0af9c0`.
A gate then found two comments wrapped mid-sentence, in `cmd/lym/main.go` and `TestRenderOverview`'s doc comment.
Both are rewritten one sentence per line in a further commit.
That rewrite left the second line of the `cmd/lym/main.go` comment unindented, which failed `TestGofmt_ModuleFilesAreFormatted`.
`lym loom fmt` fixed the indentation in a last commit.
The gate on `./internal/heddleengine` passes with 0 failures after every change, and `go vet ./cmd/lym` is clean.

## Disputed

None.

## Deferred

- The shared F2 and F4 commit (fix-proposal, LOW): both findings edit `overview_test.go`, and a split would mean rewriting commits already made.
  The F2 commit message names the F4 rows it holds.

```yaml
lens: forced-edits
outcome: reported
```

Task: forced-edits lens review of the 18-card autonomy plan.

**Findings**

1. **MEDIUM — Card 14 (D4 goto clear): one goto path never reaches the new reset.**
   - The card says every site that moves a run onto a row builds its guard through `gotoGuardEndingFixers`. That is false.
   - `internal/loomcli/arbiter.go:159` sets `GotoGuard: c.settledGuard()` directly, without `gotoGuardEndingFixers`. A goto onto PR-Review through that path keeps the recorded autonomy and session, which D4 says a goto clears.
   - The other sites do go through it: `arm.go:478`, `arbiterconsume.go:29`, `drivewiring.go:289`.
   - Fix: wrap the guard at `internal/loomcli#…arbiter` (the member that holds arbiter.go:159), or put `prReviewResettingGuard` into `settledGuard` itself. List the site on Card 14.

2. **MEDIUM — Card 10 (D4 session prompt): the stencil text left in place still addresses an operator in the pane.**
   - Card 10 moves only the "Your parent's review" and "The operator's ending" sections into the Go-chosen `review_mode` text.
   - The sections it keeps say the opposite of an autonomous session:
     - the title, `contracts/stencils/landing/landing-template-pr-review.md:12`: "discuss the pull request with the operator";
     - :19, "The operator discusses pull request … in this pane";
     - :38, "Answer the operator's questions…";
     - :47, "wait for the operator's next message";
     - :53 and :56-57.
   - Under autonomy the session would be told to wait for an operator who is not there.
   - The leading comment's marker list (:5-9) must also name `review_mode`, because `stencil.FillWith` requires every marker non-empty.
   - Fix: the Intent should name these lines and say how each becomes mode-neutral or moves into `review_mode`.

3. **LOW — Cards 7, 10, 11, 12, 13: `docs/overview.md` describes behaviour these cards change, and no card lists it.**
   - :1089: the loom verb list names `parent-review answer|parent-review await` but not the new `parent-review ask` (Card 12).
   - :1388: "It also ends Done when the operator's `lym loom pr-continue` ends the review" is wrong under a recorded autonomy (Cards 11 and 13).
   - :1385: the gate-failure wording (Card 11).
   - :1484: says `discussion_review_parent_timeout_min` sets the PR-Review ask's deadline, which `pr_review_parent_timeout_min` replaces under autonomy (Cards 7 and 9).
   - Fix: add `docs/overview.md` under `**Extends:**` or `**Edit:**` on the cards that change each behaviour.

4. **LOW — Card 1 (D1/D11): two file-level comments that name only the two old per-task keys are not listed.**
   - `internal/dobbyengine/tasksetting.go:1`: "the one reader of beam.yaml's per-task settings, pull_request and land_notice". `LookupAutonomy` joins this file.
   - `internal/dobbyengine/tasksetting_test.go:1`: "pins the per-task lookup of beam.yaml's pull_request and land_notice". Card 1 extends that test to `autonomy`.
   - Fix: list both file paths on Card 1.

5. **LOW — Card 15 (D6): the Paths registration is only partly listed.**
   - `internal/pawlengine#Deps.pathsSeams` (`internal/pawlengine/deps.go:85-91`) lists the Paths fields a producer requires, and `DrainMarker` is not in it.
   - If Card 15 instead means `NewBeamAwait` to require the field on its own, the Intent should say so, because adding it to `pathsSeams` would make every pawl producer require it.
   - `internal/hubgeom/dobbygeom_test.go:189` enumerates the paths `PawlPaths` fills. Card 15 changes `PawlPaths` but neither lists that test nor plans an assertion on the new `DrainMarker` path.

6. **LOW — Card 10: one reflective seam test is not named.**
   - `internal/loomcli#TestLandingDeps_EveryFieldPopulated` (`internal/loomcli/landingdeps_test.go:46`, marked `//lym:seam internal/landingshed#PullRequestDeps`) walks every `PullRequestDeps` field.
   - Card 10 adds `Autonomy`, `AskPRReview` and `PRReviewStanding`, so this test must see them filled. The card lists `pullRequestDepsGuardedBy` but neither this test nor its reason line.

**Tool calls beyond my context**
- `lym quarry resolve` on the plan's glyphs (twice: once to learn the output shape, once to resolve them all). The plan copied them from subagent summaries. All resolved except `internal/pawlengine#pathsSeams`, which is the method `Deps.pathsSeams`.
- A grep for references to `parentReviewRequestPrefix` and `RenderPRReviewPrompt`, the stencil marker registry, `pathsSeams`, tests of the notify sites, and skill and doc mentions. I needed the callers and forced texts the exploration lacked.
- A read of the PR-Review stencil's header and its operator lines, the prepush notice assertions, `docs/overview.md` mentions, the drift and loom config tests, and the comment headers of `tasksetting.go` and `doc.go`. I needed the texts the changes make wrong.
- A read of `newPrePushCheck`'s notify, the seam waiver lists, the pawl recipe field lists, every goto guard construction site and the `//lym:seam` markers. I needed the registrations and goto sites.
- A read of `TestLandingDeps_EveryFieldPopulated`, `pawlPathFields`, the overview lines at 1383-1487 and the `PawlPaths` test. I needed to confirm the findings above.

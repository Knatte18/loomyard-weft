```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches its branch and would fail if the code were wrong, and whether each new test is new or repeats an existing one.

**Card 5 — board-get-record (D10)**
- MEDIUM. `TestGetRecord` is planned as a new top-level tier-0 test on `runCLI`/`runJSON`, but the CLI fixture those helpers need only exists inside `internal/boardcli_test#TestCLI`.
  - `TestCLI` is tier 1 (`tiergate.Require(t, 1)`). It sets `BOARD_SKIP_GIT` and builds `cliFixture{cwd: seedCwd(t)}`, which `reset` uses (`internal/boardcli/cli_test.go:97-100`, `:75-83`).
  - `--body` is already covered there as the steps `stepGetBody` and `stepGetBodyRoundTrip` (`cli_test.go` steps list).
  - As planned, the new test has no fixture, and the card names no `seedCwd`/`cliFixture` helper.
  - Its `StoreDecisionRecord` call opens a Board without `BOARD_SKIP_GIT`, so it spawns a detached `board sync` (`internal/boardengine/spawn.go:39-53`).
  - Fix: make it a `stepGetRecord` step of `TestCLI` at tier 1. Open the board with `boardengine.OpenHub(filepath.Dir(cwd))`, which honours the env skip the test sets.

**Card 7 — discussion-approve-seam (D6)**
- LOW. `TestStoreDecisionRecord_Real` opens the hub board through `OpenHub`, which reads skip-git from the environment. The card does not set `BOARD_SKIP_GIT`, so every store spawns a detached `lym board sync` that outlives the test.
  - Evidence: `internal/boardengine/spawn.go:39-59`. The loomcli precedent is `t.Setenv("BOARD_SKIP_GIT", "1")` at `internal/loomcli/landingdeps_markdone_test.go:27`.
- LOW. `TestApproveSeam_FailsToBuild` gets a new row for `discussion` with a nil `Env.ApproveDiscussion`. That repeats the card's own `TestBouncerEntry_ApproveSeam` subtest `NilApproveDiscussionRefused`, which reaches the same `requireSeam` branch (`internal/shedrecipe/entries_bouncer_test.go:887-922`). Keep one of them.

**Card 8 — decision-add-copies (D8)**
- MEDIUM. The rows "inside Discussion-Review", "darn fake" and "beam's project record" all reach one branch: the fake's `pastApproval` returns false.
  - The new fake field `currentRow` is never read. `decisionVerb` takes the row from `readStatus` (`f.status`, `decision_test.go:155`), and the fake's `pastApproval` bool ignores the row it is passed.
  - So no row would fail if `decisionVerb` passed `""` or the wrong row to `pastApproval`.
  - The beam case has no fake state that represents a project record, so that row cannot show the "never copied" rule. That rule lives in `preRun`'s real `PastDiscussionApproval`, which card 6 tests.
  - Fix: make the fake's `pastApproval` record the row it receives and answer from a set of rows. Set `f.status.CurrentProducer` instead of adding `currentRow`. Drop the darn and beam rows as repeats.
- LOW. The two failed-commit rows ("past approval names `lym loom decision store`" and "not past names none") belong in `internal/loomcli#TestDecisionVerb_CommitFailureNamesWayForward` (`decision_test.go:1193-1205`).
- LOW. The `--amends-plan` landed-commit row needs `TestDecisionVerb_AmendsPlan`'s setup (`statusCommitLanded`, `seedAmendment`, `writeDecisionPlan`). It should be a row of that table, and the card lists none of those helpers under `Uses`.
- LOW. The card does not say what a nil `pastApproval` or a nil `storeRecord` means for a `decisionDeps` built by hand outside `decisionFake.deps`.

**Card 9 — decision-store-verb (D9)**
- MEDIUM. Several prescribed refusals have no row:
  - the absent committed record ("run `lym fabric commit` …");
  - the status-read or recipe-read error;
  - a parent that cannot be resolved.
- LOW. The success row needs `committedFound` and `committedCopy` seeded, because `newDecisionFake` leaves `committedFound` false (`decision_test.go:70-72`). The card does not say so, and without the seeding the row lands in the "no committed record" refusal instead.

**Card 6 — discussion-approval-row (D7)**
- MEDIUM. The branch where a DiscussionWrite row hands to a row that is not a Bouncer, which answers false, is reached by no row. `TestPastDiscussionApproval` drives only embedded recipes through the exported accessor, and no embedded recipe has that shape. A fixture through the unexported `discussionApprovalRow` would reach it.

**Card 1 — board-decision-record (D2)**
- LOW. In the run-held row, "`Revised` and every other field stay as loaded" passes even if the code stamped a revision. `newRenameBoard`'s clock is a `*time.Time` that does not move unless the test moves it. The row must advance `*now` before the store.
- LOW. Test economy: `TestDiscussedPayload` is a new top-level test. The other payload-key rules (`beam`, `front`, `on_land`) have no dedicated test either, and `internal/boardengine_test#TestUpsertFieldAllowlist` (`store_test.go:261`) is the payload-key test. The new test is acceptable as new, because it needs a stored flag that `NewStore("")` cannot produce.

Calls made:
- Read the `decisionFake` struct, its `deps()` lines and `TestDecisionVerb_CommitFailureNamesWayForward` (`decision_test.go`); my context lacked the fake's fields.
- Read `TestUpsertFieldAllowlist` and `TestBouncerEntry_ApproveSeam`; my context lacked their bodies.
- Read the boardcli CLI fixture and `TestCLI` header; my context lacked the fixture and tier of the `get` tests.
- Read `spawnSync` and grepped the `BOARD_SKIP_GIT` precedent in loomcli tests; my context lacked how a Board write spawns a sync under test.

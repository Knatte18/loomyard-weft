---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: internal/loomcli/wiring.go:568
    summary: PATTERN-named-constants violated, the review role name burler-review and the burler directory name are repeated as new literals instead of one named constant
    fix: In internal/burlerengine/review.go export the existing const reviewRolePrefix as ReviewRolePrefix (or add an exported accessor) and use it in internal/loomcli/wiring.go:568 in place of the literal "burler-review"; in internal/burlerengine/assist.go:57 replace the literal "burler" with one unexported const declared once in internal/burlerengine (for example burlerDirName) and use it in assist.go, and in review.go lines 134 and 159 where the same literal already stands.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/assist.go:100
    summary: A failed composeAssist leaves the assistant seat with blank instruction paths, so Table.Validate refuses the whole table and the round errors instead of the assistant not starting
    fix: In internal/burlerengine/assist.go assistSeat, when skipReason is non-empty give the seat Values whose instruction_1_path and instruction_2_path are non-blank (for example the chair's reviewFilesIn paths, or the report path), so Validate passes and seatengine.Run writes the not started marker. Add a row to TestEngine_Review_AssistantThatDoesNotStart in internal/burlerengine/assist_test.go with an assistant fan that lenses.yaml does not define, expecting a nil error, only the chair started and an ending that starts with not started.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/loomengine/review.go:249
    summary: An assist_reviewer_timeout_min of 0 loads and CapAssistTimeout returns a zero bound, which seatengine reads as the chair's own timeout, so the assistant runs uncapped and the chair can time out inside its wait
    fix: In internal/loomengine/review.go CapAssistTimeout, treat want == 0 as the capped room (return room with no reason) so a zero seat timeout never reaches the table; add a row to TestCapAssistTimeout in internal/loomengine/review_test.go with AssistReviewerTimeoutMin 0 asserting the returned bound equals room.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/seatengine/engine_test.go:400
    summary: TestEngine_Run_AssistantEndMarkerNamesTheEnding has no row for the wait-error ending, which the record lists among the endings the marker names
    fix: In internal/seatengine/engine_test.go add a row to the table of TestEngine_Run_AssistantEndMarkerNamesTheEnding in which the assistant's wait returns an error and the marker reads wait error followed by the error text, and a second row where a stop and a wait error together read stopped.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:567
    summary: The await-cap resolution through the keep-warm resolver has no test that makes the cap bind, so a wrong or ignored await cap passes every wiring test
    fix: In internal/loomcli/wiring_test.go add a row to the existing assist wiring test with a keep-warm cap and a review_timeout_min that make the cap bind, asserting the resulting Segments Assist Timeout or SkipReason.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/burlerengine/modelkeys_test.go
    summary: TestValidateAssistModelList is a per-helper test whose refusals the config_test.go malformed-entry rows and the tapestry plan rows already cover through LoadConfig and LoadTapestryConfig
    fix: In internal/burlerengine/modelkeys_test.go delete TestValidateAssistModelList after folding any assertion only it makes (the way-forward text, the 1-based entry index) into the malformed-entry rows of internal/loomengine/config_test.go.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: docs/overview.md:625
    summary: The overview's description of loom.yaml and tapestry.yaml keys omits the new assist reviewer model, fan and timeout keys
    fix: In docs/overview.md at the loom.yaml passage (line 625) add discussion_assist_reviewer_model and code_assist_reviewer_model, their _fan partners and assist_reviewer_timeout_min, and at the tapestry.yaml passage add plan_assist_reviewer_model and plan_assist_reviewer_fan; add the assist fan keys to the per-perch fan sentence at line 672.
---

### [BLOCKING] Repeated name literals break PATTERN-named-constants

**Location:** internal/loomcli/wiring.go:568; internal/burlerengine/assist.go:57
**Issue:** `PATTERN-named-constants` asks for one named constant, declared once and referenced everywhere, for a name that can go stale. The diff adds `"burler-review"` to `loomcli/wiring.go` although `burlerengine` already declares it as `reviewRolePrefix`; a role rename would leave the await cap resolved for a dead role with no failure. `assist.go` adds `"burler"` as a fourth occurrence of the directory name beside `review.go`'s existing ones. The rule makes any such violation blocking.

### [MEDIUM] A failed composition errors the round instead of skipping the assistant

**Location:** internal/burlerengine/assist.go:100
**Issue:** `composeAssist` returns zero `reviewFilePaths` and a reason when the fan is unknown, the instruction directory cannot be made, or a step stencil lacks an engine block. `assistSeat` still builds the seat from `reviewSeatValues`, which puts blank `instruction_1_path` and `instruction_2_path` into `Values`. `Table.Validate` refuses a blank seat value (`seatengine/table.go:241`) before `Run` reaches the `SkipReason` branch, so `Review` returns an error and the chair never starts. D1 and D2 say the step goes on without the assistant. The one test of an unstarted assistant covers a skip reason and an engine whose seat stencil lacks a block, neither of which fails composition. I confirmed that `Run` calls `Validate` first (`seatengine/engine.go:254`).

### [MEDIUM] A zero assist timeout escapes the cap

**Location:** internal/loomengine/review.go:249
**Issue:** Config load refuses only a negative `assist_reviewer_timeout_min`, so 0 loads. `CapAssistTimeout` returns `want` of 0 with no reason when room remains, `AssistTimeout` 0 reaches `Seat.Timeout`, and `seatSpec` reads a zero seat timeout as the table's, the chair's. The assistant's start, run and one await cap can then exceed the chair's timeout, which D1 and D3 forbid.

### [LOW] Wait-error ending has no test row

**Location:** internal/seatengine/engine_test.go:400
**Issue:** The record and the summary name the wait error among the endings. `TestEngine_Run_AssistantEndMarkerNamesTheEnding` has rows for every other ending, none for `endingWaitError`, so a regression in that branch of `endingOf` passes.

### [LOW] Await-cap resolution is untested

**Location:** internal/loomcli/wiring.go:567
**Issue:** The wiring tests pin only the default bound, which fits under the default chair timeout. The keep-warm lookup and its fallback to `DefaultAwaitCap` therefore change no test result.

### [LOW] Per-helper test duplicates public-surface rows

**Location:** internal/burlerengine/modelkeys_test.go
**Issue:** `TestValidateAssistModelList` tests a helper that `LoadConfig` and `LoadTapestryConfig` reach, and the `config_test.go` malformed-entry rows and the tapestry plan rows already pin those refusals there.

### [LOW] Overview omits the new config keys

**Location:** docs/overview.md:625
**Issue:** The overview lists loom's and tapestry's keys, including each reviewer fan key, and the diff adds eight keys without touching `docs`.

## Rejected

- lens:code-tests F1 (no `CheckMarkers` test for the assistant pass-over): the row `an assistant whose engine lacks an engine block its stencil includes is passed over` sits in `TestTable_CheckMarkers` at `internal/seatengine/table_test.go:344`.
- lens:code-correctness F3 (`run_timeout_min` of 0 with `review_timeout_min` of 0 gives the assistant no time): the record fixes this derivation of the chair's timeout, and the fork did not establish that a 0 `run_timeout_min` means unbounded.
- Handler check of the hold-notice quiet path: `seatSpec` sets `QuietHold` for every seat that is neither chair nor required, so an assistant's held turn end is silent; nothing to flag.
- Handler check of the review range for declared PATTERN changes: the constraints section lists none, and the earlier parts landed the entries the record names; nothing to flag.

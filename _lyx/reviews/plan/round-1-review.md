---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-review-refusals-name-status.md
    summary: No planned test reaches reviewState.preRun's slug-given choice of status way, so D5's core rule (choose by slug, not by where the verb runs) can be wired backwards with every listed test still green
    fix: In card 10, add `plan:internal/loomcli#TestParentReviewExchange_RewriteReviewedThenCapRejectHalts` (created by card 5) to the Edit list, and add to its Intent's Tests paragraph that, before arrival 1 opens round 1, the test runs `review notify <slug>` through the real CLI from the prime and asserts the refusal names `lyx batten status <slug>` and `from the prime`, then runs `review notify` with no slug from the task worktree and asserts it names `lyx loom status` and no `lyx batten status`; this drives preRun's `reviewSlugArg(cmd.Name(), args) != ""` choice, which the direct `reviewStatusWay("task", true/false)` calls in `TestReviewVerbs_NoOpenRequest` and `TestReviewApprove_SupersedeRefusals` bypass.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-gate-recheck-throttle.md
    summary: TestGatePending_IdleRecheckThrottled is a new top-level test for a cadence the same card already makes TestAttachGated_Pending and TestGatePending_VerdictBetweenTicksIsNotMasked assert, and its one other claim (an arrival inside the window evaluates at once) fits TestGatePending_RejectAfterPendingRepromptsOnce
    fix: In card 12, drop the Create bullet `plan:internal/shuttleengine#TestGatePending_IdleRecheckThrottled` and the `cmd/lyx/testdata/test-budget.yaml` Edit bullet together with the sentence raising the shuttleengine row (card 13 still raises it for its own test); add `internal/shuttleengine#TestGatePending_RejectAfterPendingRepromptsOnce` to the Edit list, and in the Tests paragraph say that it asserts the gate is not called on the idle tick one poll after the pending evaluation and is called on the tick its `STOP:turn2` arrival lands, two fake-clock seconds later, inside the throttle window.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-parent-review-no-pass-at-cap.md
    summary: TestResolveGateSpec_ParentReviewElementRefusals is a new table of gate-list build refusals, the behavior TestResolveGateSpec_Rejections already covers as a table keyed on the row name and an error substring; card 6 then adds to the duplicate
    fix: >-
      In card 5, drop the Create bullet `plan:internal/shedrecipe#TestResolveGateSpec_ParentReviewElementRefusals` and add `internal/shedrecipe#TestResolveGateSpec_Rejections` to the Edit list; its Intent says the table builds each row over `parentReviewEnv(t)`, which extends `newTestEnv`, and gains rows for `pass_on_cap: true` and `pass_on_cap: false` on a `parent-review` element, each also asserting `way forward:`. In card 6, replace the Edit bullet `plan:internal/shedrecipe#TestResolveGateSpec_ParentReviewElementRefusals` with `internal/shedrecipe#TestResolveGateSpec_Rejections` (the `parent-review` then `discussion` row, asserting `"parent-review" must be the last element` and `way forward:`) and `internal/shedrecipe#TestResolveGateSpec_ParentReviewResolvesMustPassMayHold` (its config becomes `discussion` then `parent-review`, asserting two entries with `parent-review` last), and drop the sentence that card 6 adds to card 5's table.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/01-engine-prompt-cache-lifetime.md
    summary: TestPromptCacheLifetime's unmapped-role, empty-role and invalid-default rows re-test resolvePromptCacheTTL's resolution, which TestPrepare_ThreadsSpecIntoLaunchCmds and TestPrepare_RejectedBeforeArtifacts already pin through their ttl_ rows
    fix: >-
      In card 1's Tests paragraph, cut TestPromptCacheLifetime's table to what the new method adds over the shared resolver, the parse to a duration and the passed-through error: the 5m default answers five minutes, a role mapped to `1h` answers an hour, and an invalid default answers an error naming `claude_prompt_cache_ttl`; drop the unmapped-role and empty-role rows.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-engine-prompt-cache-lifetime.md
    summary: Card 1 puts TestPromptCacheLifetime in command_test.go beside the TTL validation, but command_test.go holds no TTL test; the TTL tests and the ttlConfig helper the card names live in prepare_test.go
    fix: In card 1's Tests paragraph, replace `in command_test.go beside the TTL validation` with `in prepare_test.go beside the TTL rows and the ttlConfig helper`.
---

### [MEDIUM] D5's slug-driven choice is never reached by a test

**Location:** `_lyx/plan/10-review-refusals-name-status.md`

**Issue:** D5 settles that the review refusals choose their status command by whether a slug was given, not by where the verb runs, and its acceptance criterion names both the prime and a sibling task worktree.
Card 10 puts that choice in `reviewState.preRun`, which sets `statusWay` from `reviewStatusWay(target.WorktreeName, reviewSlugArg(cmd.Name(), args) != "")`.
Every test the card plans calls the verb bodies with a hand-built `reviewStatusWay("task", true)` or `reviewStatusWay("task", false)`, so `preRun`'s choice is never run: a `preRun` that passed the negated flag, or `location.WorktreeName` in place of the target's, passes every listed test.
`internal/loomcli/review_test.go` has no test that runs the cobra command; the CLI-level route is `parentreview_integration_test.go`, whose `newPRFixture` already runs the real verbs through `f.verb` from both the prime and the task worktree.

### [MEDIUM] Card 12's new throttle test duplicates the tests the card edits

**Location:** `_lyx/plan/12-gate-recheck-throttle.md`

**Issue:** The card already edits `TestAttachGated_Pending`, whose bare-pending case it says "asserts re-evaluation on the throttled cadence rather than every poll tick", and `TestGatePending_VerdictBetweenTicksIsNotMasked`, which asserts "the call count that cadence gives".
`TestGatePending_IdleRecheckThrottled` asserts that same cadence again over `newPendingFixture`.
Its only other claim, that a turn-end arrival between rechecks is evaluated on its own tick, is the path `TestGatePending_RejectAfterPendingRepromptsOnce` already drives: there the gate pends at the first Done (no `Send`, so the writer stays at the boundary), and the `STOP:turn2` arrival lands two one-second ticks later (`gateConfig`'s `PollIntervalMS: 1000`), inside the 5-second window.
`PATTERN-test-economy` makes that an assertion in the existing test, not a new top-level test and a budget raise.

### [MEDIUM] A second refusal table beside `TestResolveGateSpec_Rejections`

**Location:** `_lyx/plan/05-parent-review-no-pass-at-cap.md`

**Issue:** `internal/shedrecipe/entries_gate_test.go`'s `TestResolveGateSpec_Rejections` is the table of `gates`-list build refusals (empty list, duplicate name, negative attempts, unknown key, bad `pass_on_cap`, and others), each row asserting the row name and an error substring.
Card 5's `pass_on_cap`-on-`parent-review` refusal and card 6's not-last refusal are two more rows of that behavior, yet card 5 creates `TestResolveGateSpec_ParentReviewElementRefusals` for them and card 6 extends it.
The one obstacle is that `TestResolveGateSpec_Rejections` builds over `newTestEnv`, which cannot resolve a `parent-review` element.
`parentReviewEnv` is `newTestEnv` plus `ParentReview`, so switching the table to it changes no existing row.
Card 6's "builds" row is a success case that belongs in the `parent-review` resolve test, which card 14 later edits too, after card 6, so the order holds.

### [LOW] `TestPromptCacheLifetime` re-tests the TTL resolver's rows

**Location:** `_lyx/plan/01-engine-prompt-cache-lifetime.md`

**Issue:** `Claude.PromptCacheLifetime` is `resolvePromptCacheTTL` plus a parse.
Role resolution is already pinned in `internal/shuttleengine/claudeengine/prepare_test.go`: `ttl_role_missing_from_map_gets_default`, `ttl_empty_role_ignores_empty_map_key` and `ttl_map_entry_overrides_default` in `TestPrepare_ThreadsSpecIntoLaunchCmds`, and the invalid-value refusals in `TestPrepare_RejectedBeforeArtifacts`.
The new test is warranted for the new public method, but its unmapped-role and empty-role rows repeat resolver behavior that existing rows already cover.

### [NIT] Card 1 names the wrong test file

**Location:** `_lyx/plan/01-engine-prompt-cache-lifetime.md`

**Issue:** `internal/shuttleengine/claudeengine/command_test.go` holds `TestClaudeBinary`, `TestBuildLaunchCmd`, `TestValidateEffort` and similar tests, but no TTL test.
`ttlConfig` and the TTL rows are in `prepare_test.go`.

## Focus verdicts

- Card order for cards 3, 12, 13 and 14 holds.
  Card 13 uses card 12's `gateRecheckEvery` and `lastGateEvalAt`.
  Card 14 sets card 13's `GateEntry.Expired`, and it re-signs `resolveParentReviewClosure` after card 5's re-sign.
  Card 3 edits that resolver before card 5 without changing its signature.
  Card 4 uses card 1's method and card 2's renderer.
  `keep-alive-first` holds: card 5 edits a `closures.gate` that already has card 3's ping branch, and card 12 touches only `internal/shuttleengine`, so it does not affect card 5.
- Attack surface: card 3 bounds the ping by interval, cap, verdict, expiry, the hold and a zero-off interval.
  Card 12's throttle only delays a read, and card 13's probe finalizes only on its own entry's expiry, off the boundary, at most once per window.
  Card 4's lifetime-error fallback turns pings off, and card 9's settled success skips only a verdicted round's stamp.
  Each card states these bounds in its Intent.
- Reachability: card 5's `pass_on_cap: true` and `pass_on_cap: false` reach the refusal, since `configBool` admits both before the check.
  Card 6's `parent-review` then `discussion` resolves both elements under `parentReviewEnv`.
  Card 8's missing, empty and unreadable files reach `RecordVerdict` past `openUnsettled` on an open round.
  Card 9 is a success path and `TestReviewVerbs_Expired` keeps the refusal.
  None is cut off by an upstream check.
- Edit glyphs and re-sign arrows: every signature change carries an arrow (`newParentReviewConfig`, `resolveParentReviewClosure` twice, `reviewStoreErr`, `reviewWayNotAtCap`, and the five review verbs).
  File-path Edit targets are `doc.go` files, YAML or the recipe.
- Fidelity: D1 to D10 are each carried by cards 1 to 14, and D11 needs no card, as the overview says.
  Every Constraint is honored.
  No card adds scope the record does not license.
- `## verify:` runs `go build`, `go vet` under the integration, tmux and llm tags, then untagged and integration `go test ./...`, in the template's order, covering every targeted package.

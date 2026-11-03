---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/wiring.go:568
    summary: the keep-warm await cap is resolved for the bare role burler-review, but a segment's chair runs as discussion-burler-review, plan-burler-review or code-burler-review, so a per-role keep_warm_cap_roles entry never reaches the assistant's bound
    fix: In internal/burlerengine export one helper that forms the chair's strand role from a profile role prefix (the role Profile.rolePrefix(ReviewRolePrefix) gives, that is ReviewRolePrefix alone for an empty prefix, else prefix + "-" + ReviewRolePrefix), and have Profile.rolePrefix and reviewTable use it. In internal/loomcli/wiring.go replace the single awaitCap and the single loomengine.CapAssistTimeout call with one resolution per review segment. For each of Discussion-Review, Plan-Review and Code-Review, form the chair's name with agentname.Format(shortname, slug, <helper>(segment's role_prefix)) and resolve shuttleengines.KeepWarm(fabricengine.ConfigDirs(location), chair).Cap, then call loomengine.CapAssistTimeout with that cap and pass each segment its own bound and skip reason to loomSegmentSettings (change its assistTimeout and assistSkipReason parameters to per-segment values). Take each segment's role prefix from the same source the recipe rows' role_prefix key reads (discussion, plan, code), declared once as a constant, not a repeated literal. Extend TestWire_ReviewKeysReachTheEnv so a keep_warm_cap_roles entry for code-burler-review in claude.yaml changes only Code-Review's Assist.Timeout, and a keep_warm_cap_roles entry for the bare role burler-review changes none.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomengine/config.go:1250
    summary: the shared timeout-knob loop, its comment and its refusal message say 0 defers to shuttle's run_timeout_min, which is false for assist_reviewer_timeout_min where 0 takes the whole capped room
    fix: In the knob-validation loop of internal/loomengine/config.go, make the refusal message and the comment above it state the zero meaning per knob. Give each knob entry a zeroMeans string, "defer to shuttle's run_timeout_min" for discussion_timeout_min, review_timeout_min and friction_timeout_min and "take the whole room left before review_timeout_min" for assist_reviewer_timeout_min, and format the message with it. Keep the "must not be negative" fragment the config test asserts for the assist row.
---

### [BLOCKING] Keep-warm cap resolved for the wrong chair role

**Location:** internal/loomcli/wiring.go:568
**Issue:** `wire` forms the chair's name from `burlerengine.ReviewRolePrefix` alone and passes it to `shuttleengines.KeepWarm`.
The chair's real strand role is `SeatRole(p.rolePrefix(ReviewRolePrefix), "")`, which `internal/burlerengine/profile.go:75` leads with the row's `role_prefix`.
The shipped recipes set `role_prefix` to `discussion`, `plan` or `code` on every Burler row (`contracts/recipes/motif-recipe.yaml:44`, `contracts/recipes/darn-recipe.yaml:138`), so no chair ever runs as plain `burler-review`.
`claudeengine.keepWarmCapFor` looks the cap up by role in `keep_warm_cap_roles`, so an operator who sets a cap for `code-burler-review` has the chair's `lym shuttle await` run under that cap while `CapAssistTimeout` subtracts the default cap instead.
The assistant's bound then misses the chair's real timeout by the difference: a longer role cap lets the assistant outlast the chair's room, and a shorter one wastes it.
`PATTERN-keep-warm-cap` requires the cap to come from the resolver for the caller's role, and the caller here is the segment's chair, so this is a violation of that entry.
One cap for all three segments is wrong for the same reason, since each segment's chair is a different role.
`TestWire_ReviewKeysReachTheEnv` makes the cap bind, but only through the global `keep_warm_cap`, so it cannot see the role mismatch.

### [LOW] Timeout-knob message says zero defers to shuttle for the assist key

**Location:** internal/loomengine/config.go:1250
**Issue:** The knob-validation loop covers `assist_reviewer_timeout_min` beside the other timeout keys, and its comment and refusal message say 0 defers to shuttle's `run_timeout_min`.
For the assist key, 0 takes the whole room left before `review_timeout_min`, so the message gives an operator who sets a negative value the wrong meaning of 0.

## Rejected

- The repeated `"assist"` directory name at `internal/burlerengine/assist.go:57` and the `instructionDir("assist")` prefix at line 130 are two separate directories under the burler directory and need not agree, so they are not a stale-name hazard under `PATTERN-named-constants`; `ReviewRolePrefix` and `burlerDirName` are each declared once in `internal/burlerengine/review.go` and referenced everywhere else.
- A zero `assist_reviewer_timeout_min` cannot reach `Seat.Timeout`: `CapAssistTimeout` returns the whole room for a zero bound, and returns zero only together with a skip reason, which keeps the seat from starting.
- A failed assistant composition (unknown fan, uncreatable directory, step stencil without an engine block) returns placeholder paths from `unstartedAssist`, so `Table.Validate` passes and `Review` goes on with a nil error; `TestEngine_Review_AssistantThatDoesNotStart` covers it.
- `docs/overview.md` lists every assist model, fan and timeout key the config templates add.
